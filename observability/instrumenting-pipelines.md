---
title: Instrumenting Pipelines
description: How to wire Langfuse credentials into Yosoi pipelines, override session ids, and propagate tags.
faqs:
  - q: When should I use TelemetryPolicy vs env vars?
    a: >-
      Env vars for CI, containers, and anything where credentials live in a
      secret manager (the default LANGFUSE_* names are picked up automatically).
      TelemetryPolicy for inline scripts where you want the secret references explicit
      at the call site, or for tests where you want to construct the policy object
      directly without touching the process environment.
  - q: Can I run multiple Pipeline objects in one process with different Langfuse projects?
    a: >-
      No. obs.configure(...) initializes a singleton client; the second
      Pipeline() short-circuits on the first-call sentinel and reuses whatever
      credentials the first one configured. If you genuinely need to write to
      two projects from one process, run them in subprocesses with different env
      vars.
  - q: Why is my session id not the UUID4 I expected?
    a: >-
      If YOSOI_SESSION_ID is set in the environment when the first Pipeline()
      is constructed, the lazy resolver picks that up instead of generating a
      UUID4. Common surprises include a stale --session-id flag from a previous
      CLI run that exported the var into a parent shell, a .env that sets
      YOSOI_SESSION_ID for orchestration, or a long-lived worker process where
      an earlier batch's session id stuck.
  - q: Does setting --workers greater than 1 change how I read traces?
    a: >-
      Slightly. With workers=1 you get one trace per URL and that's the whole
      picture. With workers > 1 you also get a detached enqueue span per
      dispatch carrying count, workers, and origin. Filter by session_id in the
      Langfuse UI to see the dispatch span next to the per-URL traces; do not
      expect them to nest.
---

Yosoi pipelines are instrumented by default; the only thing you control is *where* the traces go.

## Configure via `TelemetryPolicy`

```python
import yosoi as ys

policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(
        telemetry=ys.TelemetryPolicy(
            langfuse_public_key_ref=ys.SecretRef.env('LANGFUSE_PUBLIC_KEY'),
            langfuse_secret_key_ref=ys.SecretRef.env('LANGFUSE_SECRET_KEY'),
            langfuse_host='http://localhost:3000',
        ),
    ),
)
pipeline = ys.Pipeline(policy=policy, contract=YourContract)
```

Equivalent env-var setup (preferred for CI / container deploys):

```bash
LANGFUSE_PUBLIC_KEY=pk-...
LANGFUSE_SECRET_KEY=sk-...
LANGFUSE_BASE_URL=http://localhost:3000
```

When the keys are missing, observability is a silent no-op and the pipeline runs unchanged.

## Session ids

Every process invocation generates a fresh session id as a **canonical UUID4** (e.g. `3f4a9c2e-8b1d-4f7c-9e2a-5b6c7d8e9f01`). Canonical form means the id joins cleanly across Postgres, ClickHouse, and external services without prefix stripping or custom regexes. Origin labelling ("Yosoi-emitted", "CLI vs script") lives on **session tags** (`tags=['yosoi', 'cli'|'script']`), not on the id itself.

Override for orchestration. `--session-id` accepts any string, not just UUIDs:

```bash
# CLI flag: sets YOSOI_SESSION_ID for this run
yosoi --session-id batch-2025-05-02 -f urls.txt

# Environment variable: same effect, useful for subprocess fan-out
export YOSOI_SESSION_ID=batch-2025-05-02
yosoi -f urls.txt
```

Use this when an external orchestrator wants several CLI invocations to roll up under one logical session.

> **Library-mode caveat:** the `--session-id` flag sets `os.environ['YOSOI_SESSION_ID']` and never unsets it. If you embed Yosoi inside a long-lived parent process (e.g. a worker daemon that constructs `Pipeline` repeatedly), the env var sticks across runs. Manage it yourself if you need per-run scoping (e.g. `monkeypatch.setenv` in tests, or `os.environ.pop('YOSOI_SESSION_ID', None)` between batches).

## Concurrency: four dimensions

Yosoi runs concurrently in four distinct dimensions. Most readers only know about one (`--workers`); the other three matter as soon as you scale or debug.

```text
Cross-session (multi-process):
  N processes share one YOSOI_SESSION_ID
        │
        ▼
Inter-URL (workers > 1):
  one process, N URL traces in flight, each its own trace root
        │
        ▼
Intra-URL (per-field LLM fan-out):
  one URL trace, N field LLM calls under
  asyncio.gather + Semaphore(max_concurrent=5)
        │
        ▼
Per-domain write serialization (hidden):
  asyncio.Lock per (sub)domain serializes save_selectors only
```

### Knobs

| Knob | Default | Code location | Effect |
| --- | --- | --- | --- |
| `--session-id` / `YOSOI_SESSION_ID` | auto-generated UUID4 | `yosoi/utils/observability.py:process_session_id` | Group N CLI invocations under one Langfuse session (cross-session) |
| `--workers` / `Pipeline.process_urls(workers=N)` | 1 | `yosoi/core/tasks.py` `Semaphore(max_workers)` | Inter-URL concurrency: N URLs in flight inside one process |
| `DiscoveryPolicy.max_concurrent` | 5 | `yosoi/core/discovery/orchestrator.py` `Semaphore(self._max_concurrent)` | Intra-URL per-field LLM fan-out cap |
| `_domain_locks` (internal) | one lock per (sub)domain | `yosoi/core/tasks.py:28` | Serializes `save_selectors` writes (NOT the whole task) |

### Dimension 1: Cross-session orchestration

Use case: an external orchestrator (CI matrix, k8s job, shell loop) dispatches N independent `yosoi` invocations across processes/hosts and wants them grouped under one Langfuse session for combined analysis. Pattern:

```bash
for url in $(cat urls.txt); do
  YOSOI_SESSION_ID=batch-2026-05-02 yosoi --url "$url" --contract Product
done
```

Every invocation lands in session `batch-2026-05-02`; per-URL traces still carry their own (sub)domain `user_id`. **Constraint**: the env var must be set BEFORE the first `Pipeline()` is constructed in each process. The lazy resolver (`observability.process_session_id` at `pipeline.py:193`) caches on first call and does not re-read the env afterwards.

Verification: filter the Langfuse UI by `session_id=batch-2026-05-02` and you should see traces from every process under one row.

### Dimension 2: Inter-URL workers

Concurrent dispatch (`workers > 1`) sends URLs through a `taskiq.InMemoryBroker`. Today's broker runs in-process, so workers share the parent's `_PROCESS_SESSION_ID` automatically. The orchestrator:

1. Eagerly resolves the session id at the top of `process_urls`.
2. Exports it to `YOSOI_SESSION_ID` before broker dispatch (defensive scaffolding for a future Redis broker).
3. Wraps both the sequential and concurrent dispatch branches in `observability.session(sess_id, tags=['yosoi', origin])`. This is OTel baggage, not a span, so it does not become a parent span.
4. Threads `sess_id`, `origin`, and the orchestrator-computed `domain` (via `observability.normalize_user_id`) through to each worker. Each worker opens its own `observability.session(...)` + `observability.user(...)` block before calling `scrape()`, so traces emitted in the worker carry the orchestrator's session id and the per-URL user id even if a future broker breaks OTel context propagation across the message boundary.

**Why no orchestrator-level span:** the docs promise *trace per URL*. If the orchestrator opened a span around the dispatch (e.g. `enqueue`) and made it the active OTel parent, every worker's `scrape` span would nest under it, collapsing N URLs into one trace. Instead, the orchestrator emits a *detached* `enqueue` span (recorded in the exporter, but not in the active context) and each worker's `scrape <netloc><path>` is its own trace root.

```text
session_id=X (Langfuse session, NOT a span)
tags=['yosoi','cli']
  ├── enqueue                         (detached; count=N, workers=W, origin=cli)
  ├── trace 1: scrape a.example.com/1 (root, user_id=a.example.com)
  ├── trace 2: scrape b.example.com/1 (root, user_id=b.example.com)
  ├── trace 3: scrape a.example.com/2 (root, user_id=a.example.com)
  └── trace 4: scrape b.example.com/2 (root, user_id=b.example.com)
```

In the Langfuse UI, filtering by `session_id` gives you all four traces in one view. Filtering by `user_id` gives you per-domain slices that cut across runs.

The kwargs path was chosen over W3C TraceContext middleware because (a) it works without taskiq middleware, (b) it serialises cleanly into the broker message, and (c) it's straightforward to test with `taskiq.InMemoryBroker`. W3C TraceContext propagation is documented as a future cleanup if/when a Redis broker lands.

### Dimension 3: Intra-URL per-field fan-out

Inside one URL's `discover` stage, the orchestrator runs `asyncio.gather` of N field-discovery coroutines under an `asyncio.Semaphore(max_concurrent)`. With the default `max_concurrent=5` and a contract of 5 fields, all 5 LLM calls run in parallel; with a 10-field contract they run in waves of 5.

Tune via `DiscoveryPolicy.max_concurrent`:

```python
policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(discovery=ys.DiscoveryPolicy(max_concurrent=3)),
)
pipeline = ys.Pipeline(policy=policy, contract=YourContract)
```

The orchestrator emits its `orchestrator_discover_selectors` span with `field_count` and `max_concurrent` attributes so reviewers see the planned fan-out width without timestamp arithmetic. When all fields are statically overridden, the span carries `bypass='all_overrides'` and the gather is skipped entirely.

### Dimension 4: Per-domain write serialization (hidden)

Two URLs that share a (sub)domain (e.g. `shop.example.com/a` and `shop.example.com/b`) acquire the same `asyncio.Lock` from `_domain_locks` before writing the selector cache. The lock wraps **only `save_selectors`**, not the whole task, so the URLs run concurrently up to the save step, then serialize there. URLs without a host (`file://`, `data:`) all share the empty-string lock key.

This is rarely something you tune; it exists to prevent corrupt selector files when same-domain URLs race. Worth knowing when debugging "why does my second URL on the same domain look slower?" in the Langfuse UI: look for a tail latency on `save` that doesn't appear on the first URL of a domain.

## How tags propagate

| Level | Tags | Set by |
| --- | --- | --- |
| Session | `['yosoi', 'cli']` or `['yosoi', 'script']` | Pipeline detects whether the call came from the CLI (`origin='cli'`) or the Python API (`origin='script'`, the default). |
| Trace | `[(sub)domain]` | `Pipeline.scrape()` extracts the domain from each URL and tags the per-URL trace with it. |

Custom tags can be attached at the eval layer; see [Evals & tagging](/observability/evals-and-tagging/).

## What gets traced

For each URL, the trace tree contains one root span (`scrape <netloc><path>`) plus stage spans:

- `fetch`: raw HTML retrieval (with retry events on bot detection)
- `clean`: HTML noise reduction
- `discover`: LLM selector discovery
- `verify`: selector verification against the cleaned HTML
- `extract`: content extraction with verified selectors
- `validate`: Pydantic contract validation
- `save`: selector + content persistence

LLM calls inside `discover` are instrumented natively by `pydantic-ai` and appear as nested spans with the prompt and response payload.

## FAQs

<details>
<summary>When should I use TelemetryPolicy vs env vars?</summary>

Env vars for CI, containers, and anything where credentials live in a secret manager. `Policy.from_env()` reads the default `LANGFUSE_*` names automatically. Use `TelemetryPolicy` for inline scripts where you want secret references explicit at the call site, or for tests where you want to construct the policy object directly without touching the process environment.

</details>

<details>
<summary>Can I run multiple Pipeline objects in one process with different Langfuse projects?</summary>

No. `obs.configure(...)` initializes a singleton client; the second `Pipeline()` short-circuits on the first-call sentinel and reuses whatever credentials the first one configured. If you genuinely need to write to two projects from one process, run them in subprocesses with different env vars.

</details>

<details>
<summary>Why is my session id not the UUID4 I expected?</summary>

If `YOSOI_SESSION_ID` is set in the environment when the first `Pipeline()` is constructed, the lazy resolver picks that up instead of generating a UUID4. Common surprises: a stale `--session-id` flag from a previous CLI run that exported the var into a parent shell; a `.env` that sets `YOSOI_SESSION_ID` for orchestration; or a long-lived worker process where an earlier batch's session id stuck (see the library-mode caveat above).

</details>

<details>
<summary>Does setting --workers greater than 1 change how I read traces?</summary>

Slightly. With `workers=1` you get one trace per URL and that's the whole picture. With `workers > 1` you also get a detached `enqueue` span per dispatch carrying `count`, `workers`, and `origin`. Filter by `session_id` in the Langfuse UI to see the dispatch span next to the per-URL traces; do not expect them to nest.

</details>

## See also

- [Langfuse quickstart](/observability/langfuse-quickstart/): get the keys.
- [Reading traces](/observability/reading-traces/): find the trace you just created.

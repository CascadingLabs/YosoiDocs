---
title: Evals & Tagging
description: Run mocked or live Yosoi evals tagged for point-in-time views in Langfuse, using the helpers in `yosoi.utils.observability`.
faqs:
  - q: Why use obs.session instead of langfuse.propagate_attributes directly?
    a: >-
      obs.session (and obs.user, obs.span, obs.warning, obs.flush) are no-ops
      when Langfuse keys are not configured. The same eval script runs in CI
      without a Langfuse instance, in dev against a local stack, and in prod
      against cloud Langfuse, with no conditionals around the observability
      calls. Drop down to langfuse.propagate_attributes directly only if you
      need a Langfuse-specific attribute that Yosoi's helpers don't surface.
  - q: Can I add tags inside an already-open session?
    a: >-
      Yes. obs.session is a thin wrapper around langfuse.propagate_attributes,
      which composes; nesting another propagate_attributes(tags=[...]) block
      inside merges its tags with the outer block's for spans emitted inside
      the inner block. The eval_demo script does exactly this: an outer
      obs.session for the run-level identity, an inner
      propagate_attributes(tags=['regression']) for the slice tag.
  - q: How do I tag specific stages within a single trace?
    a: >-
      Tags propagate via propagate_attributes, which scopes by block, not by
      span. To highlight one stage, emit a span event from inside it
      (obs.warning('selector_quality_low', score=0.4) adds an event on the
      current span; current_span.set_attribute('eval.severity', 'high') adds a
      queryable attribute). Filter or sort by the attribute in the Langfuse UI.
  - q: My evals run but no traces appear. What's wrong?
    a: >-
      Two things to check. (1) Agent.instrument_all() must run before any
      Pipeline() is constructed in the same process; otherwise pydantic-ai
      spans never reach the global tracer that Langfuse installed.
      obs.configure(...) calls Agent.instrument_all() itself, so this is rarely
      a problem when you go through Pipeline. (2) Short-lived scripts can exit
      before the OTel batch exporter flushes; always call obs.flush() before
      the process exits.
  - q: Why does field_agent[root] always look non-overlapping?
    a: >-
      It's added to the discovery gather list as a separate task and only runs
      after the first 5 field tasks release semaphore slots. With
      max_concurrent=5 and a 6-field contract, that's the structural reason you
      see 5 overlapping spans plus 1 trailing one in live traces. Raise
      DiscoveryConfig.max_concurrent if you want the root field to overlap too.
---

Tags are how you carve out point-in-time views in Langfuse. Yosoi sets two tags on every session out of the box (`yosoi`, `cli`/`script`) plus the (sub)domain tag on every trace. Beyond that, you attach your own tags from eval workflows so smoke runs, regression sweeps, and integration runs are filterable independently.

What follows is the *Yosoi* way to do it: drive a real `Pipeline`, lean on `yosoi.utils.observability` (which wraps Langfuse + OTel for you), and only drop down to the raw Langfuse SDK when you genuinely need to.

## The pattern

1. Build a real `Pipeline` with your contract.
2. Wrap the run in `obs.session(...)` and `obs.user(...)` (or pass extra `tags=` to either) so per-trace metadata flows down through pydantic-ai's spans.
3. Optionally use `pydantic-ai`'s `TestModel` / `FunctionModel` so no real LLM cost is incurred.
4. Filter by your tag in the Langfuse UI to read the slice.

The Yosoi helpers live in `yosoi.utils.observability` (aliased as `obs` throughout the codebase). They are no-ops when `LANGFUSE_PUBLIC_KEY` / `LANGFUSE_SECRET_KEY` aren't set, so the same script runs unchanged with telemetry on or off.

## A worked example (Yosoi-flavored)

```python
# eval_regression.py
import asyncio
from pydantic_ai import Agent
from pydantic_ai.models.test import TestModel

from yosoi.core.discovery.config import LLMConfig
from yosoi.core.pipeline import Pipeline
from yosoi.models.defaults import NewsArticle
from yosoi.utils import observability as obs


async def run() -> None:
    # Deterministic LLM stub (no provider key, no cost).
    llm_config = LLMConfig(
        provider='groq',
        model_name='llama-3.3-70b-versatile',
        api_key='unused-test-key',
        temperature=0.0,
    )
    Agent.instrument_all()  # so pydantic-ai spans land in Langfuse via OTel.

    pipeline = Pipeline(llm_config, contract=NewsArticle, quiet=True)

    # Override the inner agent's model with TestModel for deterministic evals.
    inner_agent = pipeline.discovery._agent._agent
    with inner_agent.override(model=TestModel()):
        # obs.session sets session_id + tags for every span emitted inside the block.
        # obs.user sets user_id (the per-(sub)domain identity).
        with obs.session('eval-regression-2026-05-02', tags=['yosoi', 'eval', 'regression']):
            with obs.user('shop.example.com'):
                await pipeline.scrape('https://shop.example.com/products')

    obs.flush()  # flush before the script exits so traces actually ship.


if __name__ == '__main__':
    asyncio.run(run())
```

What's Yosoi-specific here, beyond just "we used Langfuse":

- `obs.session(...)` / `obs.user(...)` are thin Yosoi context managers over `langfuse.propagate_attributes`. They give you a single import surface and they no-op cleanly when keys aren't set, so the same eval script is safe to run in CI without a Langfuse instance.
- `Pipeline.scrape(url)` produces the canonical Yosoi span tree (`scrape <netloc><path>` → `fetch` / `clean` / `discover` / `verify` / `extract` / `validate` / `save`) automatically. You don't author spans by hand.
- `pipeline.discovery._agent._agent.override(model=TestModel())` is the supported way to swap in a deterministic LLM for an eval without touching production config.

For the multi-URL flow with concurrency, see `scripts/eval_demo.py` in the Yosoi repo: it stubs the fetcher, isolates the selector cache in a tempdir, and exercises `Pipeline.process_urls(urls, workers=N)`.

### Running it

```bash
uv run python eval_regression.py
```

### Verifying the tag filter

```bash
npx -y langfuse-cli api traces list --tags regression --limit 5
```

Sample response (truncated):

```json
{
  "data": [
    {
      "id": "fc53590e3591c89ded9e6e7963ca6786",
      "name": "scrape shop.example.com/products",
      "tags": ["eval", "regression", "yosoi"],
      "userId": "shop.example.com",
      "sessionId": "eval-regression-2026-05-02",
      "observations": ["root", "fetch", "clean", "discover", "verify", "extract", "validate", "save"]
    }
  ],
  "meta": { "totalItems": 1 }
}
```

In the Langfuse UI, the same filter (`tag = regression`) shows the trace tree; clicking through reveals the per-stage span detail. Compare run-over-run to catch selector regressions or LLM-cost drift.

## Why a "detached" enqueue span (Yosoi-specific)

When you scale to `workers > 1`, Yosoi's orchestrator emits an `enqueue` span via `obs.detached_span(...)` rather than `obs.span(...)`. The difference matters:

- `obs.span(name)` calls `start_as_current_span`, which makes the new span the active OTel parent. Any span emitted inside its scope nests under it.
- `obs.detached_span(name)` calls `start_span` against a context with the active span cleared. The span is recorded by the exporter, but it does NOT become the parent of subsequent spans.

That second helper is the trick that lets the orchestrator log dispatch metadata (count, workers, origin) at the session level *without* collapsing N URL traces into one giant trace. Each worker still gets its own root `scrape <netloc><path>` trace; the `enqueue` span sits beside them in the session view, not above them.

```python
# yosoi/core/tasks.py (sketch)
with obs.detached_span('enqueue', count=len(urls), workers=workers, origin=origin):
    pass  # nothing nests inside; this span just records the dispatch.
```

If you're building your own orchestrator on top of Yosoi and want the same shape, reach for `obs.detached_span` whenever you want a span that *records* but does not *parent*.

## Concurrent example

When you run the pipeline with `workers > 1`, each URL still produces its own root trace; the orchestrator does NOT open a parent span around the dispatch (see [Instrumenting pipelines](/observability/instrumenting-pipelines/) for why). Filtering by `session_id` collapses the concurrent run into one view; filtering by `user_id` slices per-domain across runs.

The bundled `scripts/eval_demo.py` ships a `--workers N` flag that exercises this:

```bash
YOSOI_SESSION_ID=phase3-live-final uv run python scripts/eval_demo.py --workers 2
```

Add `--live` to swap `pydantic-ai` `TestModel` for a real OpenRouter call (default `openrouter:openai/gpt-4o-mini`, requires `OPENROUTER_KEY`, ~$0.20 cost ceiling per run):

```bash
YOSOI_SESSION_ID=phase4-live-final uv run python scripts/eval_demo.py --workers 2 --live
```

Internals: real `Pipeline`, 4 URLs across 2 (sub)domains (`a.example.com/1,2` and `b.example.com/1,2`). The fetcher is stubbed so the demo doesn't hit the network. Selector storage is routed through a fresh `tempfile.mkdtemp()` so the user's `.yosoi/` is never touched and every run sees a cold cache (cold cache → real per-field fan-out → visible LLM spans).

Verifying with three queries against the local Langfuse instance:

```bash
# Session-level view: all 4 URL traces under one session.
npx -y langfuse-cli api traces list --session-id phase3-live-final --limit 10
# → totalItems: 4
#   scrape a.example.com/1   user=a.example.com
#   scrape a.example.com/2   user=a.example.com
#   scrape b.example.com/1   user=b.example.com
#   scrape b.example.com/2   user=b.example.com

# Per-domain slice: filter by user_id within the session.
npx -y langfuse-cli api traces list --user-id a.example.com --session-id phase3-live-final
# → totalItems: 2

npx -y langfuse-cli api traces list --user-id b.example.com --session-id phase3-live-final
# → totalItems: 2
```

Each trace has its own `id` (one trace = one URL); URLs from different (sub)domains never collide on `user_id`. This is per-URL trace isolation in live data, not just unit tests.

### What you see when you click into a trace

A representative live run captured against the bundled local stack (project `Yosoi`, session `phase3-live-final`) produced one trace per URL plus the orchestrator's `enqueue` span:

```
phase3-live-final
├── enqueue                        (count=4, workers=2, origin=script)   trace 5da8ec89…
├── scrape a.example.com/1         user=a.example.com                    trace cbb33a2d…
├── scrape a.example.com/2         user=a.example.com                    trace 824b0876…
├── scrape b.example.com/1         user=b.example.com                    trace 7db6e475…
└── scrape b.example.com/2         user=b.example.com                    trace a162520d…
```

Opening any URL trace in the UI you see the full nested observation tree:

```
scrape a.example.com/1                               (root, input=contract, output=selectors)
├── fetch
├── clean
├── discover
│   └── orchestrator_discover_selectors
│       ├── field_agent[root]
│       │   ├── agent run                            (pydantic-ai)
│       │   │   └── chat llama-3.3-70b-versatile    (TestModel-backed; deterministic prompt + response)
│       │   └── cache_hit[headline]
│       ├── cache_hit[author]
│       ├── cache_hit[body_text]
│       ├── cache_hit[date]
│       └── cache_hit[related_content]
├── verify
├── extract
├── validate
└── save
```

The trace header's **Input** panel shows the data contract used for this scrape (Pydantic schema name + per-field descriptions + any manual selector overrides) and the **Output** panel shows the derived selectors and a sample of the validated extraction. Both are populated by `obs.set_trace_input(span, payload)` and `obs.set_trace_output(span, payload)` inside `Pipeline.scrape()`; they wrap `langfuse.observation.input` / `langfuse.observation.output` and JSON-encode the payload with `default=str` so Pydantic models, datetimes, etc. serialize without raising. The `chat <model>` LLM span carries the GenAI semantic-convention attributes (`gen_ai.system`, `gen_ai.request.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`) and the prompt/response messages.

## Concurrency dimensions (live)

A live OpenRouter run captured against the local stack with `phase4-cost-final` (two consecutive `--workers 2 --live` invocations sharing the same `YOSOI_SESSION_ID`):

```bash
YOSOI_SESSION_ID=phase4-cost-final uv run python scripts/eval_demo.py --workers 2 --live
YOSOI_SESSION_ID=phase4-cost-final uv run python scripts/eval_demo.py --workers 2 --live
```

`npx -y langfuse-cli api traces list --session-id phase4-cost-final` returned `totalItems: 10`: 2 detached `enqueue` spans (one per invocation) + 8 URL traces (4 per invocation). Both runs land in one logical Langfuse session (Dimension 1 / cross-session verified in live data).

Drilling into one URL trace (`scrape b.example.com/1`):

- `orchestrator_discover_selectors` span attributes: `field_count=6, max_concurrent=5, url=https://b.example.com/1` (Dimension 3 / intra-URL self-describing).
- 6 `field_agent[*]` spans (`headline`, `author`, `date`, `body_text`, `related_content`, `root`).
- The first 5 of 6 started within **1 ms** of each other (`23:12:38.302Z–303Z`) and overlapped throughout, real concurrent fan-out under the semaphore. 14 of 15 possible pairs overlapped (`field_agent[root]` is the lone non-overlapping span: it's added to the gather list as a separate task and runs after the first 5 release semaphore slots).
- 6 `agent run` spans (one per field) and 6 `chat openai/gpt-4o-mini` spans (one per field's LLM call) emitted by `pydantic-ai` instrumentation.
- Trace input panel populated with the `NewsArticle` contract spec; output panel populated with `path='fresh'` and the derived selectors.
- Total trace latency: 2.76 s.

> **Cost note (local stack):** Langfuse local DOES compute `totalCost`, but only when the chat span's model name matches an entry in the local `models` table. The built-in entries cover canonical names (`gpt-4o-mini`) with exact-match patterns; provider-prefixed names (`openai/gpt-4o-mini` from OpenRouter, etc.) won't match. Register a custom pricing entry once via `POST /api/public/models` to enable cost computation; see [Langfuse quickstart / Cost tracking on a non-standard model name](/observability/langfuse-quickstart/) for the recipe. After registering, fresh traces show `totalCost > 0`. Verified with `session=phase4-cost-final`: 4 of 10 traces showed non-zero cost summing to **$0.004601** for the full 2-invocation run.

For the four-dimension model behind these knobs see [Instrumenting pipelines](/observability/instrumenting-pipelines/) / Concurrency. For how to read overlapping timestamps in the UI see [Reading traces](/observability/reading-traces/) / How to read concurrency.

## Suggested tag taxonomy

| Tag | When to apply |
| --- | --- |
| `regression` | Nightly / CI run against a curated URL list to catch selector breakage. |
| `integration` | End-to-end run against a real Langfuse + real LLM, gated to a small URL set. |
| `smoke` | Fast sanity check before merging a Pipeline change. |
| `eval` | Baseline tag for *any* mocked-eval run, regardless of intent. |

Stack them: a regression run is also an eval, so `tags=['eval', 'regression']` is fine.

## Why tags, not separate sessions

Sessions are per-process. If your eval driver runs five different test cases sequentially, they all share one process and therefore one session id. Tags let each *case* keep its own filterable identity inside that session.

## FAQs

<details>
<summary>Why use obs.session instead of langfuse.propagate_attributes directly?</summary>

`obs.session` (and `obs.user`, `obs.span`, `obs.warning`, `obs.flush`) are no-ops when Langfuse keys aren't configured. The same eval script runs in CI without a Langfuse instance, in dev against a local stack, and in prod against cloud Langfuse, with no conditionals around the observability calls. Drop down to `langfuse.propagate_attributes` directly only if you need a Langfuse-specific attribute that Yosoi's helpers don't surface.

</details>

<details>
<summary>Can I add tags inside an already-open session?</summary>

Yes. `obs.session` is a thin wrapper around `langfuse.propagate_attributes`, which composes; nesting another `propagate_attributes(tags=[...])` block inside merges its tags with the outer block's for spans emitted inside the inner block. The eval_demo script does exactly this: an outer `obs.session` for the run-level identity, an inner `propagate_attributes(tags=['regression'])` for the slice tag.

</details>

<details>
<summary>How do I tag specific stages within a single trace?</summary>

Tags propagate via `propagate_attributes`, which scopes by *block*, not by span. To highlight one stage, emit a span event from inside it (`obs.warning('selector_quality_low', score=0.4)` adds an event on the current span; `current_span.set_attribute('eval.severity', 'high')` adds a queryable attribute). Filter or sort by the attribute in the Langfuse UI.

</details>

<details>
<summary>My evals run but no traces appear. What's wrong?</summary>

Two things to check: (1) `Agent.instrument_all()` must run before any `Pipeline()` is constructed in the same process; otherwise pydantic-ai spans never reach the global tracer that Langfuse installed. `obs.configure(...)` calls `Agent.instrument_all()` itself, so this is rarely a problem when you go through `Pipeline`. (2) Short-lived scripts can exit before the OTel batch exporter flushes; always call `obs.flush()` before the process exits.

</details>

<details>
<summary>Why does field_agent[root] always look non-overlapping?</summary>

It's added to the discovery gather list as a separate task and only runs after the first 5 field tasks release semaphore slots. With `max_concurrent=5` and a 6-field contract, that's the structural reason you see 5 overlapping spans plus 1 trailing one in live traces. Raise `DiscoveryConfig.max_concurrent` if you want the root field to overlap too.

</details>

## References

<a id="ref-1"></a>△ **Langfuse `npx` CLI**. Langfuse. *Command-line evals & dataset runners.* https://langfuse.com/docs

## See also

- [Reading traces](/observability/reading-traces/): once your tagged run lands, navigate to it.
- [Instrumenting pipelines](/observability/instrumenting-pipelines/): built-in session / domain tags.

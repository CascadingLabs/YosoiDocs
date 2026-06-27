---
title: Configuration
description: Environment variables and runtime options.
faqs:
  - q: "What happens if I set multiple provider keys?"
    a: "Yosoi picks one based on a built-in fallback order (Groq, Gemini, Cerebras, OpenAI, OpenRouter). To control which provider and model are used, set YOSOI_MODEL to a provider:model string (e.g. groq:llama-3.3-70b-versatile)."
  - q: "Can I change the .yosoi/ storage location?"
    a: "Not currently. The directory is always created in the working directory where Yosoi is run."
  - q: "Is .yosoi/ safe to commit to version control?"
    a: "The selector cache is safe to commit if you want to share discovered selectors across a team. The logs/, debug_html/, and content/ subdirectories are noisy and should stay gitignored."
  - q: "How do I enable debug HTML snapshots?"
    a: "Pass --debug when running the CLI. Snapshots are saved to .yosoi/debug_html/ and are useful for diagnosing extraction failures."
  - q: "Can multiple CLI invocations share one Langfuse session?"
    a: "Yes. Pass --session-id (or set YOSOI_SESSION_ID= in the environment) so every invocation under that orchestrator rolls up into one logical session in the Langfuse UI."
---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GROQ_KEY` | One of these | Groq<sup>[△](#ref-1)</sup> API key |
| `GEMINI_KEY` | One of these | Google Gemini<sup>[○](#ref-2)</sup> API key |
| `OPENAI_KEY` | One of these | OpenAI<sup>[◑](#ref-3)</sup> API key |
| `CEREBRAS_KEY` | One of these | Cerebras<sup>[◇](#ref-4)</sup> API key |
| `OPENROUTER_KEY` | One of these | OpenRouter<sup>[★](#ref-5)</sup> API key |
| `YOSOI_MODEL` | Optional | Default model in `provider:model` format (e.g. `groq:llama-3.3-70b-versatile`). Read by `Policy.from_env()`. |
| `YOSOI_FORCE` | Optional | Truthy value forces rediscovery instead of replaying the cached contract. Read into `ScrapePolicy.force`. |
| `YOSOI_FETCHER_TYPE` | Optional | Default fetch tier: `auto`, `simple`, `headless`, `headful`, `waterfall`. Read into `ScrapePolicy.fetcher_type`. |
| `YOSOI_SELECTOR_LEVEL` | Optional | Default selector ceiling (e.g. `all`, `css`, `xpath`, `role`). Defaults to `all`; read into `ScrapePolicy.selector_level`. |
| `YOSOI_DISCOVERY_MODE` | Optional | Discovery mode: `auto`, `static`, `mcp`. Read into `DiscoveryPolicy.mode`. |
| `YOSOI_CROSS_ORIGIN_DOM` | Optional | Truthy value opts browser fetchers into cross-origin DOM access (see [Cross-origin DOM access](#cross-origin-dom-access)). Read into `ScrapePolicy.cross_origin_dom`. Default off. |
| `YOSOI_LOG_LEVEL` | Optional | Logging level: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `ALL` (default: `DEBUG`) |
| `YOSOI_SESSION_ID` | Optional | Override the auto-generated Langfuse session id for this process. Equivalent to the `--session-id` CLI flag. |
| `LANGFUSE_PUBLIC_KEY` | Optional | Langfuse<sup>[⬡](#ref-6)</sup> project public key. Enables observability when set together with the secret key. |
| `LANGFUSE_SECRET_KEY` | Optional | Langfuse project secret key. |
| `LANGFUSE_BASE_URL` | Optional | Langfuse host. Defaults to `https://cloud.langfuse.com`. Set to `http://localhost:3000` for the bundled self-hosted stack. Read into `TelemetryPolicy`. |

These are the most commonly used provider keys. Yosoi supports [25+ providers](/reference/helpers/) -- each with its own environment variable. You only need one.

## Local Storage

Yosoi stores all state in `.yosoi/` in your project root (gitignored by default):

```
.yosoi/
  selectors/     # Cached selector JSON per domain
  logs/          # Run logs (run_YYYYMMDD_HHMMSS.log)
  debug_html/    # Extracted HTML snapshots (--debug only)
  content/       # Extracted output files (JSON, CSV, etc.)
  stats.json     # Cumulative LLM call and usage statistics
```

## Observability

Yosoi ships first-class [Langfuse](https://langfuse.com)<sup>[⬡](#ref-6)</sup> integration. Set `LANGFUSE_PUBLIC_KEY` and `LANGFUSE_SECRET_KEY` (plus optional `LANGFUSE_BASE_URL`) to start exporting traces. Without them, observability is a silent no-op and the pipeline runs unchanged.

The mapping is deliberate: **one process = one session, one URL = one trace, the (sub)domain = the user_id**. So filtering by user in the Langfuse UI gives you "everything we've ever scraped on `shop.example.com`", and filtering by session narrows it to a single run. Subdomains are intentionally distinct -- `shop.example.com` does not roll up into `example.com`.

For the full picture (boot script, Python config, span tree, eval tagging) see the [Observability section](/observability/).

## Policy

`ys.Policy` is the public configuration surface. It is a frozen, serializable tree that can include model selection, scrape behavior, discovery settings, telemetry, output formats, downloads, crawl policy, and atom trust settings. Raw secrets are not stored in the policy artifact. Use `ys.SecretRef.env(...)` to point at a secret and let Yosoi resolve it into a runtime-only `ResolvedRunSpec`.

```python
import yosoi as ys

policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(
        model=ys.ModelPolicy.from_string(
            'groq:llama-3.3-70b-versatile',
            credential_ref=ys.SecretRef.env('GROQ_KEY'),
        ),
        scrape=ys.ScrapePolicy(
            force=False,
            fetcher_type='auto',
            selector_level=ys.SelectorLevel.XPATH,
        ),
        discovery=ys.DiscoveryPolicy(max_concurrent=3),
        telemetry=ys.TelemetryPolicy(
            langfuse_public_key_ref=ys.SecretRef.env('LANGFUSE_PUBLIC_KEY'),
            langfuse_secret_key_ref=ys.SecretRef.env('LANGFUSE_SECRET_KEY'),
            langfuse_host='http://localhost:3000',
        ),
        output=ys.OutputPolicy(formats=('jsonl',), quiet=False),
    ),
)

rows = await ys.scrape(url, YourContract, policy=policy)
```

`Policy.from_env()` reads the environment variables above, including `YOSOI_MODEL`, `YOSOI_FORCE`, `YOSOI_DISCOVERY_MODE`, `YOSOI_ATOM_READS`, `YOSOI_ATOM_TRUST`, and Langfuse settings. `Policy.cascade(...)` merges layers from lowest to highest precedence, so a call-site policy can override env defaults without mutating global state.

### Providing the API key

There are two ways to give a model its key, and **neither stores the raw secret in the policy** -- it never appears in `model_dump()`, `repr()`, or `policy_hash`:

- **Env-resolved (recommended for deployments)** -- point at an environment variable with `ys.SecretRef.env('GROQ_KEY')`. The key is read only when you call `resolve_run_spec()`:

  ```python
  policy = ys.Policy(
      model=ys.ModelPolicy.from_string(
          'groq:llama-3.3-70b-versatile',
          credential_ref=ys.SecretRef.env('GROQ_KEY'),
      ),
  )
  spec = policy.resolve_run_spec()  # reads GROQ_KEY from os.environ
  ```

- **Direct (for a key you already hold)** -- pass `api_key=` to `from_string(...)` (or any provider helper such as `ys.groq(...)`). It is kept runtime-only, so `resolve_run_spec()` needs no environment mapping:

  ```python
  policy = ys.Policy(
      model=ys.ModelPolicy.from_string('groq:llama-3.3-70b-versatile', api_key=my_key),
  )
  spec = policy.resolve_run_spec()  # no env dict needed
  ```

  Passing a raw `{'GROQ_KEY': ...}` mapping to `resolve_run_spec()` is reserved for tests and tooling that need to resolve against a synthetic environment -- prefer one of the two forms above in application code.

### Cross-origin DOM access

By default, browser fetchers cannot run JavaScript inside a cross-origin iframe that Chrome isolates out-of-process (e.g. an embedded `google.com` frame). Set `ScrapePolicy.cross_origin_dom=True` (or the `YOSOI_CROSS_ORIGIN_DOM` env var) to launch Chrome with site-isolation field trials disabled so frame-scoped evaluation can reach those origins. Requires VoidCrawl ≥ 0.3.5.

```python
policy = ys.Policy(
    model=ys.ModelPolicy.from_string('groq:llama-3.3-70b-versatile', api_key=my_key),
    scrape=ys.ScrapePolicy(fetcher_type='headless', cross_origin_dom=True),
)
rows = await ys.scrape(url, YourContract, policy=policy)
```

This is **opt-in and off by default** because it weakens the browser's security isolation for the whole session -- only enable it when you actually need to read or drive an isolated cross-origin frame. The simple (non-browser) tier ignores it.

## Discovery concurrency

Per-field LLM fan-out within one URL is capped by an `asyncio.Semaphore`. The default cap is 5; tune it with `DiscoveryPolicy`:

```python
policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(discovery=ys.DiscoveryPolicy(max_concurrent=3)),
)
pipeline = ys.Pipeline(policy=policy, contract=YourContract)
```

| Field | Type | Range | Default | Effect |
| --- | --- | --- | --- | --- |
| `DiscoveryPolicy.max_concurrent` | `int` | 1-50 | 5 | Caps how many per-field LLM calls fan out concurrently within one URL via `asyncio.gather` + `asyncio.Semaphore`. Increase for higher throughput on small contracts; decrease if you're hitting LLM rate limits or want more deterministic ordering. |

For the four-dimension concurrency model (cross-session / inter-URL / intra-URL / per-domain write), see [Instrumenting pipelines](/observability/instrumenting-pipelines/) -- Concurrency.

## FAQs

<details>
<summary>What happens if I set multiple provider keys?</summary>

Yosoi picks one based on a built-in fallback order (Groq, Gemini, Cerebras, OpenAI, OpenRouter). To control which provider and model are used, set `YOSOI_MODEL` to a `provider:model` string (e.g. `groq:llama-3.3-70b-versatile`).

</details>

<details>
<summary>Can I change the .yosoi/ storage location?</summary>

Not currently. The directory is always created in the working directory where Yosoi is run.

</details>

<details>
<summary>Is .yosoi/ safe to commit to version control?</summary>

The selector cache is safe to commit if you want to share discovered selectors across a team. The `logs/`, `debug_html/`, and `content/` subdirectories are noisy and should stay gitignored.

</details>

<details>
<summary>How do I enable debug HTML snapshots?</summary>

Pass `--debug` when running the CLI. Snapshots are saved to `.yosoi/debug_html/` and are useful for diagnosing extraction failures.

</details>

<details>
<summary>Can multiple CLI invocations share one Langfuse session?</summary>

Yes. Pass `--session-id <id>` (or set `YOSOI_SESSION_ID=<id>` in the environment) so every invocation under that orchestrator rolls up into one logical session in the Langfuse UI.

</details>

## References

<a id="ref-1"></a>△ **Groq API**. Groq, Inc. *Low-latency LLM inference.* https://console.groq.com/docs/

<a id="ref-2"></a>○ **Gemini API**. Google. *Gemini language model API.* https://ai.google.dev/gemini-api/docs

<a id="ref-3"></a>◑ **OpenAI API**. OpenAI. *GPT model API.* https://platform.openai.com/docs/

<a id="ref-4"></a>◇ **Cerebras API**. Cerebras Systems. *High-speed LLM inference on wafer-scale hardware.* https://inference-docs.cerebras.ai/

<a id="ref-5"></a>★ **OpenRouter**. OpenRouter. *Unified API for LLM providers.* https://openrouter.ai/docs

<a id="ref-6"></a>⬡ **Langfuse**. Langfuse. *Open-source LLM observability for production AI.* https://langfuse.com/docs

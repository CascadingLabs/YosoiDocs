---
title: Reading Traces
description: Navigate the Langfuse UI for Yosoi pipelines, filter by user, slice by session, and drill into stage spans.
---

Once your pipeline has run with Langfuse credentials configured, traces appear in the **Traces** tab of your project. Here's how to read them.

## The trace list

Each row is one URL processed by `Pipeline.scrape(url)`. The columns you care about:

- **Name**: `scrape <netloc><path>`. Scannable at a glance.
- **User**: the (sub)domain. Click to filter every trace for that user.
- **Session**: the process id. Click to filter every URL processed by that one CLI / script run.
- **Tags**: `['yosoi', 'cli'\|'script']` from the session, plus the domain tag from the trace.

## Filtering: which slice do you want?

| Question | Filter |
| --- | --- |
| "Everything we've ever done on `shop.example.com`" | `user_id = shop.example.com` |
| "This batch's worth of `shop.example.com` work" | `user_id = shop.example.com` AND `session_id = <run id>` |
| "All CLI runs (vs scripted ones)" | `tag = cli` |
| "Latest scripted runs" | `tag = script` and sort by time |
| "All eval regression runs" | `tag = regression` (added by your eval workflow) |

There is **no built-in eTLD+1 aggregation**. `shop.example.com` traces will not roll up into `example.com` automatically; that's intentional, see [the model overview](/observability/).

## Drilling into a trace

Click any row to open the trace detail. You'll see the stage span tree:

```text
scrape shop.example.com/products
├── fetch          (HTTP retry events as span events)
├── clean          (HTML size before / after as attributes)
├── discover       ⇣
│   └── chat_completion  (pydantic-ai LLM span: prompt + response)
├── verify
├── extract
├── validate
└── save
```

Each stage span has a `url` attribute matching the row. The LLM span inside `discover` carries the full prompt / response body, plus token usage, plus model name, useful for debugging selector quality issues directly from the trace.

## What an LLM span looks like

The `discover` stage span has a child emitted by `pydantic-ai`'s native instrumentation:

```text
scrape shop.example.com/products
└── discover
    └── orchestrator_discover_selectors
        └── agent run               ← pydantic-ai root, attrs: model_name, gen_ai.operation.name=invoke_agent,
            │                          gen_ai.usage.input_tokens, gen_ai.usage.output_tokens,
            │                          pydantic_ai.all_messages, final_result
            └── chat <model>        ← per-LLM-call child, attrs: gen_ai.system, gen_ai.request.model,
                                       gen_ai.input.messages, gen_ai.output.messages, gen_ai.usage.*
```

Click the `chat <model>` span to see the full prompt + response messages and per-call token counts. Compare across runs to debug selector quality regressions or cost drift directly from the trace.

## How to read concurrency in a trace

Langfuse's trace detail view is **hierarchical, not Gantt**. Sibling observations stack vertically; the layout does NOT imply they ran sequentially. If you see four `field_agent[*]` spans nested under `orchestrator_discover_selectors`, the rendering doesn't tell you whether they ran in parallel or one after another. To know which, you must read the **start / end timestamps** on each observation.

The Yosoi orchestrator runs per-field LLM calls in parallel via `asyncio.gather` + `asyncio.Semaphore(max_concurrent)` (see [Instrumenting pipelines](/observability/instrumenting-pipelines/) / Dimension 3). To verify in a trace:

1. Open `orchestrator_discover_selectors`. Read its `field_count` and `max_concurrent` attributes; that tells you the planned fan-out width without timestamp arithmetic.
2. Hover over each `field_agent[<name>]` child observation; Langfuse shows `startTime` and `endTime`.
3. Two observations overlap in time when one's `startTime < other.endTime` AND `other.startTime < one.endTime`. That's parallel execution.

### Worked example

A live trace from `phase3-live-final` (4 fields, default `max_concurrent=5`) had:

```
field_agent[headline]    start=2026-05-02T21:42:05.114  end=…05.987
field_agent[author]      start=2026-05-02T21:42:05.117  end=…05.992
field_agent[date]        start=2026-05-02T21:42:05.119  end=…06.012
field_agent[body_text]   start=2026-05-02T21:42:05.122  end=…05.998
```

All four start within 8ms of each other and finish within ~25ms; they ran in parallel under the semaphore. With `TestModel` (deterministic stub) the windows are tight; with a real OpenRouter LLM call the windows widen to multi-hundred-ms and the parallelism is unmistakable.

If you instead see start times offset by hundreds of ms across siblings, fan-out is happening but the LLM provider is rate-limiting individual calls (lower `max_concurrent` to reduce thrash, or check provider quotas).

## Concurrent runs (workers > 1)

When you run with `workers > 1`, each URL still produces its own root `scrape <netloc><path>` trace; there is intentionally no orchestrator-level parent span. The orchestrator does emit a *detached* `enqueue` span (count, workers, origin) so the dispatch metadata is recorded, but it is NOT in the active OTel context and does not parent worker traces. See [Instrumenting pipelines](/observability/instrumenting-pipelines/) / Dimension 2 for why.

**Filter by `session_id` to collapse a concurrent run into a single view**; that's what session ids are for. Filtering by `user_id` cuts across concurrent runs to surface every trace for one (sub)domain.

## When something goes wrong

- **Bot detection retries**: appear as span events on `fetch` with the indicator list and status code.
- **Verification partial-fails**: `verify` ends successfully but the span attributes show `<verified>/<total>` field counts. Drill into `discover` to see the LLM's reasoning.
- **AI discovery exhausted retries**: `discover` ends with a `warning` event `'All AI attempts failed'` and the trace status is error.

## FAQ

### A trace is missing from the list. Where did it go?

Most common causes, in order: (1) the run completed before the OTel batch exporter flushed, so the spans never shipped (call `obs.flush()` at the end of short-lived scripts); (2) the keys point at a different project than the one you're viewing in the UI (check the project switcher); (3) the URL had no host (`file://`, `data:`), which is logged but produces no `user_id` and may surprise UI filters that scope by user. Start at `session_id`, not `user_id`, when a trace seems missing.

### My LLM span shows tokens but `totalCost` is `0`. Why?

The model name on the `chat <model>` span doesn't match an entry in the Langfuse `models` pricing table. Cloud Langfuse covers most canonical names; local Langfuse only ships exact patterns for the canonical set, so provider-prefixed names (`openai/gpt-4o-mini` from OpenRouter) won't match until you register them. See [Langfuse quickstart / Cost tracking on a non-standard model name](/observability/langfuse-quickstart/) for the recipe. Existing traces are NOT recomputed after you register a new pricing entry.

### Can I export trace data for offline analysis?

Yes. `npx -y langfuse-cli api traces list --session-id <id> --limit 1000` returns JSON you can pipe into `jq` or load into a DataFrame. For per-span detail, fetch each trace by id with `npx -y langfuse-cli api traces get <trace_id>`. The CLI hits the same public API the UI uses, so anything visible in the UI is exportable.

### How do I find the slowest stage in a trace?

Open the trace and read the duration column on each stage span. There is no per-stage histogram in the UI itself; for that, run a SQL query against ClickHouse (the data is in the `observations` table with `start_time` / `end_time` columns) or aggregate via the CLI. As a quick eyeball: `discover` is almost always the dominant stage when there's an LLM cost; `fetch` dominates when bot detection is retrying.

## See also

- [Instrumenting pipelines](/observability/instrumenting-pipelines/): how the spans get created.
- [Evals & tagging](/observability/evals-and-tagging/): point-in-time slices via custom tags.

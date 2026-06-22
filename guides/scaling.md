---
title: Scaling
description: Running Yosoi at scale with queues, observability, and distributed storage.
faqs:
  - q: "What is the first scaling knob I should use?"
    a: "Start with --workers or Pipeline.process_urls(..., workers=N). Increase slowly while watching target-site rate limits and LLM-provider limits."
  - q: "Do I need Redis or RabbitMQ to run batches?"
    a: "No. Current concurrent processing runs inside one Python process. External queues are planned for multi-machine orchestration."
  - q: "How do I share selector discoveries across machines today?"
    a: "Share the .yosoi/selectors/ directory through your deployment artifact or storage layer. Native distributed selector storage is still roadmap work."
---

Yosoi scales today by combining local selector caches, concurrent workers, browser fetchers, and observability. Larger distributed caches and external queue integrations are planned, but they are not required for ordinary batch scraping.

## Current Support

- Use `Pipeline.process_urls(..., workers=N)` or the CLI `--workers` flag for concurrent URL processing.
- Use `.yosoi/selectors/` as the local selector cache and `.yosoi/fetch/` as the learned fetcher strategy cache.
- Use [Observability](/observability/) when you need traces for discovery, extraction, and model behavior.
- Keep one shared fetcher instance per batch when you are writing Python orchestration code.

## Planned Integrations

The integrations below are planned or in progress:

| Integration | Role |
|-------------|------|
| **Redis**<sup>[△](#ref-1)</sup> | Distributed selector cache and job state |
| **RabbitMQ**<sup>[○](#ref-2)</sup> | URL queue and worker coordination |
| **Prefect**<sup>[◑](#ref-3)</sup> | Workflow orchestration and scheduling |
| **Langfuse**<sup>[◇](#ref-4)</sup> | LLM observability and prompt tracing |
| **Persistence** | Durable result storage across runs |
| **Turso**<sup>[★](#ref-5)</sup> | Embedded distributed SQLite for selector snapshots |

Treat these as roadmap items until a guide documents a concrete configuration.

## FAQs

<details>
<summary>What is the first scaling knob I should use?</summary>

Start with `--workers` or `Pipeline.process_urls(..., workers=N)`. Increase slowly while watching target-site rate limits and LLM-provider limits.

</details>

<details>
<summary>Do I need Redis or RabbitMQ to run batches?</summary>

No. Current concurrent processing runs inside one Python process. External queues are planned for multi-machine orchestration.

</details>

<details>
<summary>How do I share selector discoveries across machines today?</summary>

Share the `.yosoi/selectors/` directory through your deployment artifact or storage layer. Native distributed selector storage is still roadmap work.

</details>

## References

<a id="ref-1"></a>△ **Redis**. Redis Ltd. *In-memory data structure store used as a database, cache, and message broker.* https://redis.io/docs/

<a id="ref-2"></a>○ **RabbitMQ**. Broadcom. *Open-source message broker supporting multiple messaging protocols.* https://www.rabbitmq.com/docs/

<a id="ref-3"></a>◑ **Prefect**. Prefect Technologies. *Workflow orchestration platform for data and ML pipelines.* https://docs.prefect.io/

<a id="ref-4"></a>◇ **Langfuse**. Langfuse. *Open-source LLM observability, tracing, and analytics platform.* https://langfuse.com/docs

<a id="ref-5"></a>★ **Turso**. ChiselStrike. *Embedded distributed SQLite built on libSQL.* https://docs.turso.tech/

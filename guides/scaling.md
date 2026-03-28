---
title: Scaling
description: Running Yosoi at scale with queues, observability, and distributed storage.
---

*Full documentation coming soon.*

Yosoi is designed to scale beyond a single machine. The integrations below are planned or in progress:

| Integration | Role |
|-------------|------|
| **Redis** | Distributed selector cache and job state |
| **RabbitMQ** | URL queue and worker coordination |
| **Prefect** | Workflow orchestration and scheduling |
| **Langfuse** | LLM observability and prompt tracing |
| **Persistence** | Durable result storage across runs |
| **Turso** | Embedded distributed SQLite for selector snapshots |

More integrations TBD.

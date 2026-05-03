---
title: Langfuse Quickstart
description: Get Langfuse keys (cloud is recommended; self-hosted is for the adventurous) and wire them into Yosoi.
faqs:
  - q: Cloud or self-hosted, which should I pick?
    a: >-
      Cloud unless you have a specific reason not to. The free tier covers
      individual and small-team usage, the cost-tracking model table is broader,
      and there is no infrastructure to maintain. Self-host when you have
      data-residency requirements, want an offline dev loop, or are trialling
      Langfuse before signing up.
  - q: How do I change the localhost port?
    a: >-
      The default docker-compose.langfuse.yml exposes the web UI on 3000. To use
      a different host port, edit the langfuse-web service's ports mapping (e.g.
      "3300:3000") and update LANGFUSE_BASE_URL in your .env to match. The
      container-side port stays 3000; only the host-side mapping and your client
      URL change. Restart the stack with docker compose -f
      docker-compose.langfuse.yml down && up -d.
  - q: My local stack runs but no traces appear in the UI. What broke?
    a: >-
      Three usual suspects, in order. (1) Keys not loaded, confirm
      LANGFUSE_PUBLIC_KEY and LANGFUSE_SECRET_KEY are in the env of the process
      running Pipeline. (2) Wrong project, keys are scoped to one project, so a
      newly created project still receives traces from old keys against the old
      project. (3) Trace not flushed, short-lived scripts can exit before the
      OTel batch exporter ships its buffer. The Yosoi CLI flushes on exit; in a
      notebook or REPL, call langfuse.flush() explicitly.
  - q: Do I need ClickHouse running locally?
    a: >-
      If you self-host, yes. The Docker Compose file boots ClickHouse
      automatically and Langfuse will not start without it. See the
      observability overview for why ClickHouse is the storage layer in the
      first place.
  - q: Can I switch from local to cloud later without losing my Yosoi config?
    a: >-
      Yes. The only env var that changes is LANGFUSE_BASE_URL. Swap it from
      http://localhost:3000 to https://cloud.langfuse.com, swap the keys, and
      Yosoi sends traces to the new destination. Existing local trace data stays
      in your local ClickHouse; it does not migrate.
---

Yosoi's observability is Langfuse. You have two ways to get a Langfuse project:

1. **Langfuse Cloud (recommended)**: a free tier, no infra, comprehensive built-in cost tracking. Sign up at [https://cloud.langfuse.com](https://cloud.langfuse.com) and follow the official [Langfuse cloud quickstart](https://langfuse.com/docs/get-started). Two-minute setup.
2. **Self-hosted (for the adventurous)**: run the full stack on your machine via Docker Compose. More moving parts, your own data, no third party. Covered below.

Either way, the wiring into Yosoi is the same three env vars; only `LANGFUSE_BASE_URL` differs.

## Cloud Langfuse (recommended)

After creating an organisation and project at [https://cloud.langfuse.com](https://cloud.langfuse.com) (or the [US region](https://us.cloud.langfuse.com)), copy the project's **Public Key** and **Secret Key** and put them in your `.env`:

```bash
LANGFUSE_PUBLIC_KEY=pk-...
LANGFUSE_SECRET_KEY=sk-...
LANGFUSE_BASE_URL=https://cloud.langfuse.com    # or https://us.cloud.langfuse.com
```

That's it. Any `Pipeline` you construct will pick them up via `TelemetryConfig` and start sending traces. Cloud Langfuse maintains a comprehensive built-in pricing table, so `totalCost` populates automatically on most providers.

For SSO, team management, and self-serve org setup, follow [Langfuse's official setup docs](https://langfuse.com/docs/get-started).

## Self-hosted Langfuse (for the adventurous)

Yosoi ships with a `docker-compose.langfuse.yml` at the repo root that runs a complete Langfuse stack locally: Postgres, Redis, ClickHouse, MinIO, and the Langfuse web/worker services.

### Boot the stack

From the Yosoi repo root:

```bash
docker compose -f docker-compose.langfuse.yml up -d
```

The web UI listens on [http://localhost:3000](http://localhost:3000). The first time you visit, create an organisation, a project, and copy the project's **Public Key** and **Secret Key**.

### Wire it to Yosoi

Set these in your `.env` (the values from your local project):

```bash
LANGFUSE_PUBLIC_KEY=pk-...
LANGFUSE_SECRET_KEY=sk-...
LANGFUSE_BASE_URL=http://localhost:3000
```

Any `Pipeline` you construct will pick them up via `TelemetryConfig` and start sending traces. Without these keys, observability is a silent no-op and pipelines run unchanged.

### Cost tracking on a non-standard model name (OpenRouter, LiteLLM, Bedrock)

Langfuse computes `totalCost` server-side by joining `gen_ai.usage.*` against a `models` table. The shipped table covers canonical names (`gpt-4o-mini`, `claude-3-5-sonnet`, etc.) with **exact** match patterns like `(?i)^(gpt-4o-mini)$`.

Providers that prefix model names (e.g. OpenRouter's `openai/gpt-4o-mini`, LiteLLM's `bedrock/anthropic.claude-3-5-sonnet-20241022-v2:0`) won't match the built-in entries. Token usage still populates on every `chat <model>` span, but `totalCost` stays 0.

Register a custom pricing entry for your prefixed name, a one-time setup per provider/model:

```bash
curl -X POST http://localhost:3000/api/public/models \
  -u "$LANGFUSE_PUBLIC_KEY:$LANGFUSE_SECRET_KEY" \
  -H 'Content-Type: application/json' \
  -d '{
    "modelName": "openai/gpt-4o-mini",
    "matchPattern": "(?i)^(openai/gpt-4o-mini)$",
    "inputPrice": 1.5e-07,
    "outputPrice": 6e-07,
    "unit": "TOKENS"
  }'
```

Pricing values from the provider's docs (USD per token). The match pattern is a Postgres regex; use `(?i)^...$` for exact case-insensitive match. Fresh traces emitted after the entry is registered will show `totalCost > 0`. Existing traces are NOT retroactively recomputed.

Cloud Langfuse maintains a more comprehensive built-in pricing table; this gotcha is local-only.

### Stopping the stack

```bash
docker compose -f docker-compose.langfuse.yml down
```

Add `-v` to also drop the persistent volumes if you want a clean slate next boot.

## FAQs

<details>
<summary>Cloud or self-hosted, which should I pick?</summary>

Cloud unless you have a specific reason not to. The free tier covers individual and small-team usage, the cost-tracking model table is broader, and there is no infrastructure to maintain. Self-host when you have data-residency requirements, want an offline dev loop, or are trialling Langfuse before signing up.

</details>

<details>
<summary>How do I change the localhost port?</summary>

The default `docker-compose.langfuse.yml` exposes the web UI on `3000`. To use a different host port (say `3300`), edit the `langfuse-web` service's `ports` mapping in `docker-compose.langfuse.yml`:

```yaml
services:
  langfuse-web:
    ports:
      - "3300:3000"   # host:container; only the left side changes
```

Then update your `.env` to match:

```bash
LANGFUSE_BASE_URL=http://localhost:3300
```

Restart the stack (`docker compose -f docker-compose.langfuse.yml down && up -d`) and visit [http://localhost:3300](http://localhost:3300). The container-side port stays `3000`; only the host-side mapping and your client URL change. If port `3000` collides with another tool (Grafana, a Next.js dev server) this is the cleanest fix.

</details>

<details>
<summary>My local stack runs but no traces appear in the UI. What broke?</summary>

Three usual suspects, in order:

1. **Keys not loaded**: confirm `LANGFUSE_PUBLIC_KEY` and `LANGFUSE_SECRET_KEY` are in the env of the process running `Pipeline`. A common miss: setting them in `.env` but not sourcing it in the shell that runs your script.
2. **Wrong project**: the keys are scoped to one project. If you created a new project in the UI, the old keys still authenticate but write to the old project. Check the project switcher.
3. **Trace not flushed**: short-lived scripts can exit before the OTel batch exporter ships its buffer. The Yosoi CLI flushes on exit; if you're in a notebook or REPL, call `langfuse.flush()` explicitly.

</details>

<details>
<summary>Do I need ClickHouse running locally?</summary>

If you self-host, yes: the Docker Compose file boots ClickHouse automatically and Langfuse won't start without it. See the [observability overview](/observability/) for why ClickHouse is the storage layer in the first place.

</details>

<details>
<summary>Can I switch from local to cloud later without losing my Yosoi config?</summary>

Yes. The only env var that changes is `LANGFUSE_BASE_URL`. Swap it from `http://localhost:3000` to `https://cloud.langfuse.com`, swap the keys, and Yosoi sends traces to the new destination. Existing local trace data stays in your local ClickHouse; it does not migrate.

</details>

## See also

- [Instrumenting pipelines](/observability/instrumenting-pipelines/): pass these keys via `TelemetryConfig`.
- [Configuration](/yosoi/configuration/): the full env-var table.

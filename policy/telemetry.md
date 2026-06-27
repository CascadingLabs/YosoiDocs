---
title: Telemetry Policy
description: Configure Langfuse references without storing raw telemetry secrets.
---

`telemetry` configures observability. It stores secret references, not raw secret values.

```yaml
telemetry:
  langfuse_public_key_ref:
    source: env
    name: LANGFUSE_PUBLIC_KEY
  langfuse_secret_key_ref:
    source: env
    name: LANGFUSE_SECRET_KEY
  langfuse_host: https://cloud.langfuse.com
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `langfuse_public_key_ref` | secret ref or null | `null` | Environment reference for the Langfuse public key. |
| `langfuse_secret_key_ref` | secret ref or null | `null` | Environment reference for the Langfuse secret key. |
| `langfuse_host` | string or null | `null` | Langfuse host. Use `http://localhost:3000` for a local stack. |

If telemetry keys are absent, observability is a no-op and Yosoi runs unchanged.

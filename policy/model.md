---
title: Model Policy
description: Select the LLM provider and model without storing raw API keys.
---

`model` controls the LLM used for selector discovery.

```yaml
model:
  provider: groq
  model_name: llama-3.3-70b-versatile
  temperature: 0.01
  max_tokens: null
  extra_params: null
  credential_ref:
    source: env
    name: GROQ_KEY
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `provider` | string or null | `null` | Provider name, such as `groq`, `gemini`, `openai`, `openrouter`, `anthropic`, or `ollama`. |
| `model_name` | string or null | `null` | Provider-specific model name. Must be set with `provider`. |
| `temperature` | number | `0.01` | Sampling temperature from `0.0` to `2.0`. Low values keep discovery deterministic. |
| `max_tokens` | integer or null | `null` | Optional model output token cap. |
| `extra_params` | object or null | `null` | Provider-specific extra model parameters. |
| `credential_ref` | object or null | `null` | Secret reference. Use `{source: env, name: GROQ_KEY}`. |

## Secrets

Do not put raw API keys in policy files. Use `credential_ref` and environment variables.

```yaml
model:
  provider: openai
  model_name: gpt-4.1-mini
  credential_ref:
    source: env
    name: OPENAI_KEY
```

If `model` is omitted, Yosoi can still pick a provider from the environment fallback order when a supported provider key is present.

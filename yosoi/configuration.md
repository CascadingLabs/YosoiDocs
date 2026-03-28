---
title: Configuration
description: Environment variables and runtime options.
---

## Environment variables

| Variable | Required | Description |
|----------|----------|-------------|
| `GROQ_KEY` | One of these | Groq API key |
| `GEMINI_KEY` | One of these | Google Gemini API key |
| `OPENAI_KEY` | One of these | OpenAI API key |
| `CEREBRAS_KEY` | One of these | Cerebras API key |
| `OPENROUTER_KEY` | One of these | OpenRouter API key |
| `LOGFIRE_TOKEN` | Optional | Enables Logfire tracing |

## Local storage

Yosoi stores all state in `.yosoi/` in your project root (gitignored by default):

```
.yosoi/
  selectors/     # Cached selector JSON per domain
  logs/          # Run logs (run_YYYYMMDD_HHMMSS.log)
  debug_html/    # Extracted HTML snapshots (--debug only)
```

## Observability

Set `LOGFIRE_TOKEN` to send traces to [Logfire](https://logfire.pydantic.dev) for cloud-based observability. Without it, logs are written locally only.

## FAQs

<details>
<summary>What happens if I set multiple provider keys?</summary>

Yosoi picks one automatically. To control which provider is used, set `YOSOI_PROVIDER` to the provider name (e.g. `groq`, `openai`).

</details>

<details>
<summary>Can I change the .yosoi/ storage location?</summary>

Not currently. The directory is always created in the working directory where Yosoi is run.

</details>

<details>
<summary>Is .yosoi/ safe to commit to version control?</summary>

The selector cache is safe to commit if you want to share discovered selectors across a team. The `logs/` and `debug_html/` subdirectories are noisy and should stay gitignored.

</details>

<details>
<summary>How do I enable debug HTML snapshots?</summary>

Pass `--debug` when running the CLI. Snapshots are saved to `.yosoi/debug_html/` and are useful for diagnosing extraction failures.

</details>

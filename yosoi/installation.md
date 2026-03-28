---
title: Installation
description: How to install Yosoi and set up your environment.
---

Yosoi requires Python 3.10+ and uses uv [[1]](#ref-1) for package management.

## Install from PyPI

```bash
uv add yosoi
```

## Install from source

```bash
git clone https://github.com/CascadingLabs/Yosoi
cd Yosoi
uv sync
```

For development tools:

```bash
uv sync --group dev
```

## API keys

Create a `.env` file in your project root (see `env.example`):

```bash
# At least one provider is required
GROQ_KEY=your_groq_api_key
GEMINI_KEY=your_gemini_api_key
OPENAI_KEY=your_openai_api_key
CEREBRAS_KEY=your_cerebras_api_key
OPENROUTER_KEY=your_openrouter_api_key

# Optional: cloud observability via Logfire
LOGFIRE_TOKEN=your_logfire_token
```

Yosoi works with any of these providers [[2]](#ref-2)[[3]](#ref-3)[[4]](#ref-4)[[5]](#ref-5)[[6]](#ref-6). You only need one. Set `LOGFIRE_TOKEN` to enable cloud observability via Logfire [[7]](#ref-7).

## FAQs

<details>
<summary>Can I use pip instead of uv?</summary>

Yes. `pip install yosoi` works fine. The docs use `uv` because it is faster and handles virtual environments automatically, but there is no hard dependency on it.

</details>

<details>
<summary>Do I need all five API keys?</summary>

No. One is enough. Yosoi will use whichever key it finds. If multiple keys are present, set `YOSOI_PROVIDER` to specify which one to use.

</details>

<details>
<summary>Where should I put my .env file?</summary>

In your project root, next to `pyproject.toml`. Yosoi loads it automatically via `python-dotenv` [[8]](#ref-8). Do not commit it to version control.

</details>

<details>
<summary>What Python versions are supported?</summary>

Python 3.10 and above. Earlier versions are not supported due to use of modern type hint syntax.

</details>

## References

<a id="ref-1"></a>1. **uv**. Astral. *Python package and project manager.* https://docs.astral.sh/uv/

<a id="ref-2"></a>2. **Groq API**. Groq, Inc. *Low-latency LLM inference.* https://console.groq.com/docs/

<a id="ref-3"></a>3. **Gemini API**. Google. *Gemini language model API.* https://ai.google.dev/gemini-api/docs

<a id="ref-4"></a>4. **OpenAI API**. OpenAI. *GPT model API.* https://platform.openai.com/docs/

<a id="ref-5"></a>5. **Cerebras API**. Cerebras Systems. *High-speed LLM inference on wafer-scale hardware.* https://inference-docs.cerebras.ai/

<a id="ref-6"></a>6. **OpenRouter**. OpenRouter. *Unified API for LLM providers.* https://openrouter.ai/docs

<a id="ref-7"></a>7. **Logfire**. Pydantic. *Cloud observability and tracing.* https://logfire.pydantic.dev/docs/

<a id="ref-8"></a>8. **python-dotenv**. Saurabh Kumar. *Reads key-value pairs from a `.env` file and sets them as environment variables.* https://pypi.org/project/python-dotenv/

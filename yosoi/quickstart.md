---
title: Quick Start
description: Install Yosoi and start scraping in minutes.
---

## Install

```bash
uv add yosoi
```

Add at least one provider key to a `.env` file in your project root:

```bash
GROQ_KEY=your_groq_api_key
# or GEMINI_KEY, OPENAI_KEY, CEREBRAS_KEY, OPENROUTER_KEY
```

## Option 1: CLI

Run discovery and extraction from the terminal without writing any Python.

```bash
uv run yosoi --url https://news.ycombinator.com
```

Yosoi fetches the page, asks the LLM to identify selectors, validates them, and saves the result to `.yosoi/`. Run the same command again and it reads from cache.

### CLI flags

| Flag | Description |
|------|-------------|
| `--url` | Target URL to discover selectors for |
| `--provider` | LLM provider to use (e.g. `groq`, `openai`) |
| `--debug` | Save extracted HTML to `.yosoi/debug_html/` |

## Option 2: Python script

Use the Python API when you need to process results programmatically.

```python
from yosoi import Yosoi

yosoi = Yosoi()
articles = yosoi.scrape("https://news.ycombinator.com")

for article in articles:
    print(article.headline, article.url)
```

Discovery only happens once per domain. Subsequent calls read from the local cache with no LLM calls and no cost.

For typed extraction with validation, define a contract:

```python
import yosoi as ys

class Article(ys.Contract):
    title: str = ys.Title()
    author: str = ys.Author()
    url: str = ys.Url()

pipeline = ys.Pipeline(ys.auto_config(), contract=Article)

async for item in pipeline.scrape('https://news.ycombinator.com'):
    print(item.get('title'), item.get('url'))
```

## FAQs

<details>
<summary>What if selector discovery fails?</summary>

Check that your API key is valid and that the target URL is publicly accessible. Run with `--debug` to capture the extracted HTML and inspect what was sent to the LLM.

</details>

<details>
<summary>How do I force re-discovery for a domain I have already cached?</summary>

Delete the corresponding file from `.yosoi/selectors/` and run again. Yosoi will treat the domain as new and run discovery from scratch.

</details>

<details>
<summary>Can I use pip instead of uv?</summary>

Yes. `pip install yosoi` works fine. The docs use `uv` because it is faster and handles virtual environments automatically, but there is no hard dependency on it.

</details>

<details>
<summary>How do I switch providers?</summary>

Pass `--provider groq` (or `openai`, `gemini`, etc.) to the CLI, or set `YOSOI_PROVIDER` in your `.env` file.

</details>

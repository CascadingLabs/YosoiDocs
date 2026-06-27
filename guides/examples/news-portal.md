---
title: News Portal
description: Extract articles from a news portal using Yosoi.
---

Target: [Mountainhome Herald](https://qscrape.dev/l1/news) (QScrape L1)

This example extracts article headlines, authors, dates, and URLs from a news portal. The page contains multiple articles, so `scrape()` yields one item per article.

## CLI

The built-in `NewsArticle` contract extracts headlines, authors, dates, body text, and related content. No custom contract needed.

```bash
uvx yosoi --url https://qscrape.dev/l1/news --contract NewsArticle --output json
```

To save only specific output formats:

```bash
uvx yosoi --url https://qscrape.dev/l1/news --contract NewsArticle --output json,csv
```

## Python

Define a custom contract when you want control over which fields are extracted.

```python
# news.py
import asyncio
import yosoi as ys

class Article(ys.Contract):
    title: str = ys.Title()
    author: str = ys.Author()
    date: str = ys.Datetime()
    url: str = ys.Url()

async def main():
    policy = ys.Policy.cascade(
        ys.Policy.from_env(),
        ys.Policy(scrape=ys.ScrapePolicy(fetcher_type='simple')),
    )

    rows = await ys.scrape('https://qscrape.dev/l1/news', Article, policy=policy)
    ys.show(rows)

asyncio.run(main())
```

Run it:

```bash
uv run python news.py
```

You can also use a custom contract from the CLI by pointing to the file:

```bash
uvx yosoi --url https://qscrape.dev/l1/news --contract news.py:Article
```

## What to Expect

- First run: Yosoi calls the LLM to discover selectors, then extracts and renders each article with `ys.show(...)`. Selectors are cached to `.yosoi/selectors/`.
- Second run: Selectors are loaded from cache. No LLM call, near-instant extraction.

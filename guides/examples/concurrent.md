---
title: Concurrent Scraping
description: Scrape multiple sites concurrently with Yosoi.
---

This example processes multiple QScrape sites in parallel using `process_urls()` with a worker pool. A Rich progress table appears automatically when `workers > 1`.

## CLI

Pass a file of URLs and set `--workers`:

```bash
# urls.txt
https://qscrape.dev/l1/news
https://qscrape.dev/l1/scoretap
https://qscrape.dev/l1/eshop
```

```bash
uv run yosoi --file urls.txt --contract NewsArticle --workers 3 --output json
```

Or pass multiple `--url` flags in sequence (processed sequentially without `--file`):

```bash
uv run yosoi --url https://qscrape.dev/l1/news --url https://qscrape.dev/l1/eshop --workers 2
```

## Python

```python
# concurrent.py
import asyncio
import yosoi as ys
from yosoi.utils.files import init_yosoi, is_initialized

URLS = [
    'https://qscrape.dev/l1/news',
    'https://qscrape.dev/l1/scoretap',
    'https://qscrape.dev/l1/eshop',
]

async def main():
    if not is_initialized():
        init_yosoi()

    pipeline = ys.Pipeline(ys.auto_config(), contract=ys.NewsArticle)
    results = await pipeline.process_urls(URLS, workers=3)

    print(f'{len(results["successful"])} succeeded, {len(results["failed"])} failed')

asyncio.run(main())
```

Run it:

```bash
uv run python concurrent.py
```

## What to expect

- With `workers=3`, all three URLs are processed simultaneously.
- A Rich Live progress table shows real-time status for each URL.
- Selector discovery runs in parallel -- if two URLs share a domain, one worker discovers while the other waits, then both use the cached result.
- The `results` dict has three keys: `"successful"`, `"failed"`, and `"skipped"`.

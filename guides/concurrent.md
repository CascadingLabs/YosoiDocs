---
title: Concurrent Scraping
description: Process multiple URLs in parallel with the Pipeline.
---

`Pipeline.process_urls()` accepts a list of URLs and a `workers` argument. When `workers > 1`, a Rich Live progress table appears automatically — no extra setup required.

## Basic usage

```python
import asyncio
import yosoi as ys
from yosoi import Pipeline
from yosoi.utils.files import init_yosoi, is_initialized

URLS = [
    'https://news.ycombinator.com',
    'https://lobste.rs',
    'https://thenewstack.io',
]

async def main() -> None:
    if not is_initialized():
        init_yosoi()

    config = ys.auto_config()  # picks up YOSOI_MODEL / provider keys from .env
    pipeline = Pipeline(config, contract=ys.NewsArticle)

    results = await pipeline.process_urls(URLS, workers=3)
    print(f'Done: {len(results["successful"])} succeeded, {len(results["failed"])} failed')

asyncio.run(main())
```

## Result shape

`process_urls()` returns a dict with two keys:

| Key | Value |
|-----|-------|
| `successful` | List of URLs that completed without error |
| `failed` | List of URLs that raised an exception |

## Notes

- Selector discovery (the one-time LLM call) is also parallelised across workers.
- Cached domains skip discovery entirely — only extraction runs.
- `workers=1` (the default) runs sequentially with no progress table.

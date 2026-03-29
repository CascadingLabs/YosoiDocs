---
title: JSON Output
description: Save Yosoi extraction results to structured files.
---

Set `output_format` on the `Pipeline` (or `--output` on the CLI) to persist extracted data automatically.

## CLI

```bash
uv run yosoi --url https://qscrape.dev/l1/news --contract NewsArticle --output json
```

Combine multiple formats in one run:

```bash
uv run yosoi --url https://qscrape.dev/l1/news --contract NewsArticle --output json,csv
```

## Python

```python
# output.py
import asyncio
import yosoi as ys

class Article(ys.Contract):
    title: str = ys.Title()
    author: str = ys.Author()

async def main():
    pipeline = ys.Pipeline(
        ys.auto_config(),
        contract=Article,
        output_format='json',
    )
    async for item in pipeline.scrape('https://qscrape.dev/l1/news'):
        print(item.get('title'))

asyncio.run(main())
```

Run it:

```bash
uv run python output.py
```

Results are written to `.yosoi/content/<domain>/results.json`. Multi-item pages are saved as `{"items": [...]}`.

## Multiple Formats at Once

```python
pipeline = ys.Pipeline(config, contract=Article, output_format=['json', 'csv'])
```

## Supported Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| `json` | `.json` | One file per domain |
| `jsonl` | `.jsonl` | One JSON object per line (append-friendly) |
| `ndjson` | `.jsonl` | Alias for `jsonl` |
| `csv` | `.csv` | Flat tabular output |
| `md` | `.md` | Markdown table |
| `xlsx` | `.xlsx` | Requires `openpyxl` (`uv add yosoi[tabular]`) |
| `parquet` | `.parquet` | Requires `pyarrow` (`uv add yosoi[tabular]`) |

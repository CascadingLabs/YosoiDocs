---
title: JSON Output
description: Save Yosoi extraction results to structured files.
---

Set `output_format` on the `Pipeline` to persist extracted data automatically.

## JSON

```python
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

Results are written to `.yosoi/content/<domain>/results.json`. Multi-item pages are saved as `{"items": [...]}`.

## Other formats

Pass a different format string or combine multiple:

```python
# Single format
pipeline = ys.Pipeline(config, contract=Article, output_format='csv')

# Multiple formats at once
pipeline = ys.Pipeline(config, contract=Article, output_format=['json', 'csv'])
```

| Format | Extension | Notes |
|--------|-----------|-------|
| `json` | `.json` | One file per domain |
| `jsonl` | `.jsonl` | One JSON object per line (append-friendly) |
| `ndjson` | `.jsonl` | Alias for `jsonl` |
| `csv` | `.csv` | Flat tabular output |
| `md` | `.md` | Markdown table |
| `xlsx` | `.xlsx` | Requires `openpyxl` (`uv add yosoi[tabular]`) |
| `parquet` | `.parquet` | Requires `pyarrow` (`uv add yosoi[tabular]`) |

From the CLI, use `--output` or `-o`:

```bash
uv run yosoi --url https://qscrape.dev/l1/news --output json,csv
```

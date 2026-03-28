---
title: Examples
description: Working Yosoi examples using QScrape test sites.
---

All examples in this documentation use [QScrape](https://qscrape.dev) sites rather than real public websites. QScrape provides stable, purpose-built scraping targets that won't break, won't rate-limit you, and won't be affected by the traffic this documentation generates.

If you want to follow along locally, every example here will work against the corresponding QScrape URL without modification.

## Extract articles from a news portal

Target: [Mountainhome Herald](https://qscrape.dev) (QScrape L1)

```python
import asyncio
import yosoi as ys

class Article(ys.Contract):
    title: str = ys.Title()
    author: str = ys.Author()
    date: str = ys.Date()
    url: str = ys.Url()

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=Article)

    async for item in pipeline.scrape('https://l1.mountainhome-herald.qscrape.dev'):
        print(item.get('title'), item.get('author'))

asyncio.run(main())
```

## Extract products from an e-commerce catalogue

Target: VaultMart (QScrape L1)

```python
import asyncio
import yosoi as ys

class Product(ys.Contract):
    root = ys.css('article.product')

    name: str = ys.Title()
    price: float = ys.Price()
    rating: str = ys.Rating()

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=Product)

    async for item in pipeline.scrape('https://l1.vaultmart.qscrape.dev'):
        print(item.get('name'), item.get('price'))

asyncio.run(main())
```

## Extract sports scores

Target: ScoreTap (QScrape L1)

```python
import asyncio
import yosoi as ys

class Match(ys.Contract):
    root = ys.css('div.match-row')

    teams: str = ys.Field(description='Team names')
    score: str = ys.Field(description='Final score')
    date: str = ys.Date()

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=Match)

    async for item in pipeline.scrape('https://l1.scoretap.qscrape.dev'):
        print(item.get('teams'), item.get('score'))

asyncio.run(main())
```

## Scrape multiple sites concurrently

```python
import asyncio
import yosoi as ys
from yosoi import Pipeline
from yosoi.utils.files import init_yosoi, is_initialized

URLS = [
    'https://l1.mountainhome-herald.qscrape.dev',
    'https://l1.scoretap.qscrape.dev',
    'https://l1.vaultmart.qscrape.dev',
]

async def main():
    if not is_initialized():
        init_yosoi()

    pipeline = Pipeline(ys.auto_config(), contract=ys.NewsArticle)
    results = await pipeline.process_urls(URLS, workers=3)

    print(f'{len(results["successful"])} succeeded, {len(results["failed"])} failed')

asyncio.run(main())
```

## Save output to JSON

```python
pipeline = ys.Pipeline(
    ys.auto_config(),
    contract=Article,
    output_format='json',
)
```

Results are written to `.yosoi/content/<domain>/results.json`.

## FAQs

<details>
<summary>Why use QScrape instead of real sites in examples?</summary>

Real sites change. A working example today might fail tomorrow if the target redesigns. QScrape sites are stable, controlled, and designed exactly for this purpose. They also avoid sending unwanted traffic to sites that did not ask for it.

</details>

<details>
<summary>Can I run these examples against real sites?</summary>

Yes. Swap any QScrape URL for a real one. Yosoi works the same way. The QScrape URLs are used here purely for documentation stability.

</details>

<details>
<summary>Do I need a QScrape account?</summary>

No. QScrape sites are publicly accessible with no authentication required.

</details>

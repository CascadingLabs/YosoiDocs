---
title: E-Commerce Catalogue
description: Extract products from an e-commerce catalogue using Yosoi.
---

Target: [VaultMart](https://qscrape.dev/l1/eshop) (QScrape L1)

This example extracts product names, prices, and ratings from a catalogue page. The contract describes the data; Yosoi discovers the selectors.

## CLI

The built-in `Product` contract extracts name, price, rating, reviews count, description, and availability:

```bash
uvx yosoi --url https://qscrape.dev/l1/eshop --contract Product --output json
```

## Python

When you omit `root`, Yosoi asks the AI to find the repeating container as part of selector discovery. If the page has a listing layout, the discovered root is cached with the rest of the selectors.

```python
import asyncio
import yosoi as ys

class Product(ys.Contract):
    name: str = ys.Title()
    price: float = ys.Price()
    rating: str = ys.Rating()

async def main():
    policy = ys.Policy.cascade(
        ys.Policy.from_env(),
        ys.Policy(scrape=ys.ScrapePolicy(fetcher_type='simple')),
    )

    rows = await ys.scrape('https://qscrape.dev/l1/eshop', Product, policy=policy)
    ys.show(rows)

asyncio.run(main())
```

Run it:

```bash
uv run python products.py
```

## If Discovery Goes Wrong

The most common failure mode is a repeating container that's too broad or too narrow. Run with `--debug` to save the extracted HTML for inspection, then improve the field descriptions or move to the advanced selector guide if you need to pin a selector.

Run with `--debug` to save the extracted HTML for inspection:

```bash
uvx yosoi --url https://qscrape.dev/l1/eshop --contract products_auto.py:Product --debug
```

Snapshots are saved to `.yosoi/debug_html/`.

## What to Expect

Each product card on the page yields one item. Prices are coerced to `float` automatically by `ys.Price()` -- currency symbols and thousands separators are stripped.

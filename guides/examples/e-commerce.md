---
title: E-Commerce Catalogue
description: Extract products from an e-commerce catalogue using Yosoi.
---

Target: [VaultMart](https://qscrape.dev/l1/eshop) (QScrape L1)

This example extracts product names, prices, and ratings from a catalogue page. It also explains when to let the AI discover the repeating container automatically versus pinning it yourself.

## CLI

The built-in `Product` contract extracts name, price, rating, reviews count, description, and availability:

```bash
uv run yosoi --url https://qscrape.dev/l1/eshop --contract Product --output json
```

## Python: Automatic Root Discovery

When you omit `root`, Yosoi asks the AI to find the repeating container as part of selector discovery. The AI sees an optional `root` field in the contract schema with the description: *"Selector for the repeating wrapper element that contains one complete item."* If the page has a listing layout, the AI returns a root selector; if it's a single-item page, it returns `null`.

```python
# products_auto.py
import asyncio
import yosoi as ys

class Product(ys.Contract):
    name: str = ys.Title()
    price: float = ys.Price()
    rating: str = ys.Rating()

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=Product)

    async for item in pipeline.scrape('https://qscrape.dev/l1/eshop'):
        print(item.get('name'), item.get('price'))

asyncio.run(main())
```

This works well for most sites. The AI inspects the HTML for repeating structural patterns and returns a selector like `article.product_pod` or `div.product-card`.

## Python: Pinned Root

Pin the container with `root` when you already know the wrapper element, or when the AI picks the wrong one:

```python
# products_pinned.py
import asyncio
import yosoi as ys

class Product(ys.Contract):
    root = ys.css('article.product')

    name: str = ys.Title()
    price: float = ys.Price()
    rating: str = ys.Rating()

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=Product)

    async for item in pipeline.scrape('https://qscrape.dev/l1/eshop'):
        print(item.get('name'), item.get('price'))

asyncio.run(main())
```

Run either version:

```bash
uv run python products_auto.py
uv run python products_pinned.py
```

Or use a custom contract from the CLI:

```bash
uv run yosoi --url https://qscrape.dev/l1/eshop --contract products_pinned.py:Product
```

## Automatic vs. Pinned Root

| | Automatic | Pinned |
|---|---|---|
| **How it works** | The AI discovers the root alongside other selectors | You set `root = ys.css('...')` on the contract class |
| **Precedence** | Used when no `root` is set on the contract | Always takes precedence -- the AI's root is discarded |
| **When to use** | You don't know the markup, or you want zero-config discovery | The AI picks the wrong container, or you need deterministic behaviour across runs |
| **Cache behaviour** | The discovered root is cached in `.yosoi/selectors/` like any other field | The pinned root is used directly; it is not stored in the cache |
| **Stale handling** | If the root selector stops matching, Yosoi forces full re-discovery | If a pinned selector stops matching, extraction fails -- you need to update the contract |

### Resolution Order

1. If the contract has `root = ys.css(...)` or `root = ys.xpath(...)`, that value is always used. The AI's discovered root (if any) is discarded.
2. If no `root` is set, Yosoi uses whatever the AI returned for the `root` field during discovery.
3. If the AI returned `null` for root (single-item page), extraction runs against the full page.

### When Automatic Discovery Goes Wrong

The most common failure mode is the AI selecting a container that's too broad (e.g. the whole page body) or too narrow (e.g. a single element within a card). Symptoms:

- **Too broad**: `scrape()` yields one giant item containing all products mashed together.
- **Too narrow**: `scrape()` yields many items but most fields are `None`.

Fix it by inspecting the page source (`Cmd+U` / `Ctrl+U`), finding the correct repeating element, and pinning it:

```python
class Product(ys.Contract):
    root = ys.css('article.product')  # pin the correct container
    ...
```

Run with `--debug` to save the extracted HTML for inspection:

```bash
uv run yosoi --url https://qscrape.dev/l1/eshop --contract products_auto.py:Product --debug
```

Snapshots are saved to `.yosoi/debug_html/`.

## What to Expect

Each product card on the page yields one item. Prices are coerced to `float` automatically by `ys.Price()` -- currency symbols and thousands separators are stripped.

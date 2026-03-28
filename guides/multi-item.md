---
title: Multi-Item Extraction
description: Extract multiple items from listing and catalog pages.
---

Yosoi handles pages where a selector matches many elements — product cards, article lists, search results — by detecting a repeating container and yielding one item per match.

## Auto-discovery

The default: pass a `Contract` and let the AI find the container.

```python
import yosoi as ys

class Product(ys.Contract):
    name: str = ys.Title()
    price: float = ys.Price()
    rating: str = ys.Rating()

pipeline = ys.Pipeline(ys.auto_config(), contract=Product)

async for item in pipeline.scrape('https://books.toscrape.com'):
    print(item.get('name'), item.get('price'))
```

`scrape()` is an async generator that yields one `ContentMap` per item found on the page.

## Pinning the container

If the AI guesses wrong, or you already know the wrapper element, set `root` on the contract:

```python
class Product(ys.Contract):
    root = ys.css('article.product_pod')

    name: str = ys.Title()
    price: float = ys.Price(hint='Includes £ symbol')
```

`root` takes precedence over whatever the AI discovers. Use this when you want deterministic behaviour across runs.

## Single-item pages

`scrape()` works the same way on detail pages — it just yields one item. No special handling needed.

```python
class BookDetail(ys.Contract):
    title: str = ys.Title()
    price: float = ys.Price(hint='Includes £ symbol')
    availability: str = ys.Field(description='Stock availability status')

async for item in pipeline.scrape('https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html'):
    print(item)  # yields exactly one item
```

## Saving output

Pass `output_format='json'` to persist results. Multi-item pages are saved as `{"items": [...]}`.

```python
pipeline = ys.Pipeline(config, contract=Product, output_format='json')
```

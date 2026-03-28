---
title: Manual Selectors
description: Pin CSS selectors on individual fields to skip AI discovery where you already know the answer.
---

Target: [Eldoria Registry of Deeds](https://qscrape.dev/l1/taxes) (QScrape L1)

Sometimes you already know the right selector for one or more fields. Yosoi lets you pin selectors per field -- pinned fields skip AI discovery entirely, while the rest are still discovered automatically. This is useful when:

- The AI consistently picks the wrong selector for a specific field
- You want deterministic extraction for critical fields
- You want to reduce LLM calls by providing known selectors upfront

## Pinning a field selector

Use `selector='...'` on any field to provide a CSS selector directly. That field is excluded from AI discovery.

```python
# taxes.py
import asyncio
import yosoi as ys

class TaxRecord(ys.Contract):
    root = ys.css('tr.record-row')

    owner: str = ys.Field(description='Property owner name', selector='td.owner')
    address: str = ys.Field(description='Property address', selector='td.address')
    assessed_value: float = ys.Price(description='Assessed property value')
    tax_due: float = ys.Price(description='Tax amount due')

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=TaxRecord)

    async for item in pipeline.scrape('https://qscrape.dev/l1/taxes'):
        print(item.get('owner'), item.get('assessed_value'))

asyncio.run(main())
```

Here `owner` and `address` use pinned selectors -- the AI never sees them. `assessed_value` and `tax_due` are discovered by the AI as normal. This mix-and-match approach lets you lock down the fields you're confident about while still getting AI help for the rest.

Run it:

```bash
uv run python taxes.py
```

Or from the CLI:

```bash
uv run yosoi --url https://qscrape.dev/l1/taxes --contract taxes.py:TaxRecord --output json
```

## Pinning on type factories

The `selector` parameter works on built-in type factories too, not just `ys.Field()`:

```python
class Article(ys.Contract):
    title: str = ys.Title(selector='h1.article-title')
    author: str = ys.Author(selector='span.byline')
    date: str = ys.Datetime()       # discovered by AI
    body: str = ys.BodyText()        # discovered by AI
```

The `selector` kwarg is forwarded to the underlying `Field()` call, so it works the same way regardless of which factory you use.

## How it works under the hood

When Yosoi builds the selector model to send to the AI, it calls `Contract.get_selector_overrides()` to find every field with a `selector` set. Those fields are excluded from the schema the AI sees -- the AI only discovers selectors for the remaining fields.

During extraction, pinned selectors are merged back in and used directly alongside the AI-discovered ones. From the extractor's perspective, there is no difference between a pinned and a discovered selector.

## Pinning the root

As covered in the [E-Commerce example](/guides/examples/e-commerce/), you can also pin the repeating container:

```python
class Product(ys.Contract):
    root = ys.css('article.product')       # pinned root (CSS)
    # root = ys.xpath('//article[@class="product"]')  # or use XPath

    name: str = ys.Title()
    price: float = ys.Price()
```

`root` accepts `ys.css()`, `ys.xpath()`, `ys.regex()`, or `ys.jsonld()`. CSS is the most common.

## Fully manual contracts

You can pin every field and skip AI discovery entirely:

```python
class FullyPinned(ys.Contract):
    root = ys.css('div.item')

    title: str = ys.Title(selector='h2.title')
    price: float = ys.Price(selector='span.price')
    rating: str = ys.Rating(selector='div.stars')
```

With every field pinned, Yosoi makes zero LLM calls. It fetches the page, applies your selectors, coerces the values through the type system, and returns the results. This is useful when you already know the markup and want Yosoi purely for its extraction pipeline and type coercion.

## Mixing strategies

A single contract can combine all of these:

```python
class HybridContract(ys.Contract):
    root = ys.css('article.listing')                        # pinned root
    title: str = ys.Title(selector='h2 a')                  # pinned field
    price: float = ys.Price()                                # AI-discovered
    description: str = ys.Field(description='Item summary')  # AI-discovered
    category: str = ys.Field(
        description='Product category',
        hint='Usually in a breadcrumb or sidebar label',     # hint guides the AI
    )
```

| Field | Source | Notes |
|-------|--------|-------|
| `root` | Pinned via `ys.css()` | Always takes precedence over AI |
| `title` | Pinned via `selector=` | Excluded from AI discovery |
| `price` | AI-discovered | Type factory provides the description |
| `description` | AI-discovered | `description` tells the AI what to look for |
| `category` | AI-discovered | `hint` gives the AI extra guidance beyond the description |

## FAQs

<details>
<summary>What is the difference between description and hint?</summary>

`description` tells the AI *what* the field contains (e.g. "Product category"). `hint` gives *where* or *how* to find it (e.g. "Usually in a breadcrumb or sidebar label"). Both are sent to the AI during discovery. Use `description` always; add `hint` when the field is ambiguous or in an unusual location.

</details>

<details>
<summary>Are pinned selectors cached?</summary>

No. Pinned selectors are defined in the contract and used directly at extraction time. They are not written to `.yosoi/selectors/`. If you change a pinned selector in your code, the new value is used immediately on the next run.

</details>

<details>
<summary>What happens if a pinned selector stops matching?</summary>

Extraction returns `None` for that field. Unlike AI-discovered selectors, pinned selectors are not re-discovered automatically -- you need to update the contract. Annotate optional fields as `T | None` if you want graceful handling.

</details>

<details>
<summary>Can I pin with XPath instead of CSS?</summary>

For `root`, yes -- use `ys.xpath('//div[@class="item"]')`. For individual fields, `selector=` currently takes a CSS selector string. If you need XPath for a specific field, leave it for AI discovery and guide it with `hint`.

</details>

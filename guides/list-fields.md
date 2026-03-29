---
title: List Fields
description: Extract multiple values per field using list[T] in your contract.
---

Declare a field as `list[T]` when a page contains several values for the same slot -- tags, prices, categories, authors. Yosoi handles two common DOM patterns automatically.

## Pattern A: Separate Elements

One selector, multiple matching nodes. The extractor collects the text from each match.

```html
<a class="tag">love</a>
<a class="tag">life</a>
<a class="tag">inspiration</a>
```

```python
import yosoi as ys

class Quote(ys.Contract):
    root = ys.css('div.quote')

    text: str = ys.Field(description='The quote text')
    author: str = ys.Author()
    tags: list[str] = ys.Field(description='Topic tags for the quote')
```

The AI discovers `a.tag` and the extractor returns all matched texts as a list.

## Pattern B: Delimited String

One selector, one node with a comma/semicolon-separated value. Yosoi splits it automatically.

```html
<span>love, life, and inspiration</span>
```

```python
raw = {'text': '"Be yourself."', 'author': 'Oscar Wilde', 'tags': ['love, life, and inspiration']}
result = Quote.model_validate(raw)
# result.tags == ['love', 'life', 'inspiration']
```

The default delimiter splits on `,`, `;`, and the word `and`.

## Custom Delimiter

Override the split pattern with `delimiter`:

```python
class Article(ys.Contract):
    title: str = ys.Title()
    categories: list[str] = ys.Field(description='Categories', delimiter=r'\s*\|\s*')
```

```python
raw = {'title': 'Some Article', 'categories': ['Tech | Science | AI']}
result = Article.model_validate(raw)
# result.categories == ['Tech', 'Science', 'AI']
```

## Per-Element Coercion

Yosoi type coercions apply element-by-element when the field is a list. Mix `list[float]` with `ys.Price()` to strip currency symbols from each value:

```python
class PriceComparison(ys.Contract):
    name: str = ys.Title()
    vendor_prices: list[float] = ys.Price(description='Prices from different vendors')
```

```python
raw = {'name': 'Widget', 'vendor_prices': ['$12.99', '£9.50', '€11.00']}
result = PriceComparison.model_validate(raw)
# result.vendor_prices == [12.99, 9.50, 11.00]
```

## Pinning the Selector

If the AI discovers the wrapper element instead of the individual items, pin the selector explicitly:

```python
tags: list[str] = ys.Field(description='Topic tags', selector='a.tag')
```

## FAQs

<details>
<summary>Can I use list[int] or list[float]?</summary>

Yes. Type coercions apply per element, so `list[float]` with `ys.Price()` strips currency symbols from each value before converting to float.

</details>

<details>
<summary>What if the delimiter varies across items on the same page?</summary>

Use a regex delimiter that covers both cases. For example, `delimiter=r'\s*[,;]\s*'` handles both commas and semicolons. If the variation is too unpredictable, a custom `Validators` method gives you full control.

</details>

<details>
<summary>What if the AI discovers the wrong selector for a list field?</summary>

Pin it with `selector='a.tag'` on the field. This bypasses discovery for that field while letting the AI handle the rest.

</details>

<details>
<summary>What is returned if no elements match?</summary>

An empty list `[]`. If you need to treat an empty list as a validation error, add a validator that checks the length.

</details>

---
title: Yahoo Finance Reuse Test
description: A live test use case for discovering a quote contract once and replaying it across same-shaped ticker pages.
---

Yahoo Finance is a useful live smoke target for page identity reuse because quote pages share a stable conceptual shape across tickers while still being real, rendered pages. Use it as an operator test, not as a deterministic docs fixture: the site can change, block traffic, or vary by region.

The Yosoi repository includes a runnable example:

```bash
uv run python examples/field_atoms/yahoo_finance_demo.py
```

The example defines a small quote contract:

```python
import yosoi as ys

class Quote(ys.Contract):
    name: str = ys.Field(description='the company name, e.g. Apple Inc.')
    price: str = ys.Field(description='the current share price, a number')
```

It then scrapes several ticker pages in one call:

```python
TICKERS = ['AAPL', 'MSFT', 'NVDA']
PAGES = [f'https://finance.yahoo.com/quote/{ticker}' for ticker in TICKERS]

policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(
        model=ys.ModelPolicy(provider='claude-sdk', model_name='claude-opus-4-7'),
        scrape=ys.ScrapePolicy(fetcher_type='headless'),
    ),
)

results = await ys.scrape(
    PAGES,
    Quote,
    policy=policy,
)
```

## Test Setup

Run this from the Yosoi repository root.

```bash
YOSOI_ATOM_READS=1 YOSOI_ATOM_TRUST=strict \
uv run python examples/field_atoms/yahoo_finance_demo.py
```

Expected operator behavior:

1. The first quote page requires normal discovery.
2. Same-shaped quote pages can reuse the discovered field atoms.
3. All returned rows contain a company name and price-like value.
4. If Yahoo changes the page or blocks the request, the test should fail visibly instead of silently accepting empty data.

## Useful Variations

Change the ticker list to include another large-cap quote page:

```python
TICKERS = ['AAPL', 'MSFT', 'NVDA', 'GOOGL']
```

Then include a page that should not share the quote shape, such as a Yahoo Finance news page. That page should not be served from the quote-page atoms; it should fall back to discovery or fail closed.

## CI Guidance

Do not make this live target a required CI test. Keep CI on fixture-based unit tests such as the Yahoo Finance shape fixtures in the Yosoi test suite. Use the live script as a manual smoke before broadening reuse behavior or changing trust policy defaults.

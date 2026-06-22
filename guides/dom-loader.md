---
title: DOMLoader
description: Drive JavaScript-heavy pages to a fully-loaded state before selector discovery.
faqs:
  - q: "When should I use the waterfall instead of the simple fetcher?"
    a: "Use waterfall when you're scraping a mix of static and dynamic pages and don't want to think about which is which. It adds latency on static pages (one failed Chrome attempt before returning the plain HTTP result) but is otherwise transparent. For known dynamic sites, use headless or headful directly."
  - q: "My page loads content but DOMLoader doesn't find it. What's wrong?"
    a: "Run with --debug to save the HTML that Yosoi sees after DOMLoader finishes. If the content is present, the issue is with selector discovery -- not loading. If the content is absent, the page likely uses a trigger pattern not covered by the current catalogues (catalogues.py). Check which patterns DOMLoader probes for and compare against what the page actually uses."
  - q: "Can I use DOMLoader without the full waterfall?"
    a: "Yes -- ScrapePolicy(fetcher_type='headless') or ScrapePolicy(fetcher_type='headful') use DOMLoader directly without the Simple HTTP tier."
  - q: "Does DOMLoader interact with authenticated pages?"
    a: "Not currently. DOMLoader probes for publicly-observable DOM patterns (cookie banners, load-more buttons). It does not handle login forms, session cookies, or authentication flows. Pass pre-authenticated HTML to Yosoi if the target requires login."
---

Yosoi's static HTML fetcher handles most sites -- content is in the server response, and CSS selectors work immediately. For pages that require JavaScript to render content (single-page apps, infinite scroll feeds, accordion-gated data), Yosoi ships a browser-backed fetcher tier that drives the page to a fully-loaded state before discovery begins.

The component that manages this is `DOMLoader`.

## What DOMLoader Does

`DOMLoader` sits inside the browser-based fetcher and runs after the initial page load. It works through a behavior tree -- a priority-ordered sequence of probes and actions -- that clears obstacles (cookie banners, modals) and exhausts content triggers (load-more buttons, pagination, infinite scroll) before capturing the final HTML.

```
navigate(url)
  ↓
DOMLoader.run(tab)
  ↓
  clear obstacles → exhaust content triggers → wait for DOM stability
  ↓
tab.content() → raw HTML
  ↓
HTMLCleaner → LLM discovery
```

The behavior tree **restarts** after every successful action. It only stops when every node in the tree returns `FAILURE` -- meaning nothing left to do.

## How the Behavior Tree Works

The tree has two levels: a `Selector` at the root tries each branch in order and returns `SUCCESS` on the first that works; a `Sequence` checks a condition then runs an action, returning `FAILURE` if the condition is false.

```
Selector
├── Sequence(HasOverlay, Selector(Sequence(HasCloseButton, ClickClose), Skip))
├── Sequence(HasTrigger(LOAD_MORE), ClickTrigger)
├── Sequence(HasTrigger(ACCORDION), ClickTrigger)
├── Sequence(HasTrigger(TAB), ClickTrigger)
├── Sequence(HasTrigger(PAGINATION), ClickTrigger)
└── Sequence(HasTrigger(INFINITE_SCROLL), Scroll)
```

**Priority order** -- obstacles fire first so a cookie banner never blocks a load-more probe:

| Kind | What it finds |
|------|---------------|
| Cookie | Consent banners with accept buttons |
| Popup | Modal dialogs with close buttons |
| Age Gate | Age verification screens |
| Load More | "Load more", "Show more" buttons |
| Accordion | Collapsed `[aria-expanded="false"]` sections |
| Tab | Unselected `[role="tab"]` panels |
| Pagination | Next-page links and `a[rel="next"]` |
| Infinite Scroll | Bottom-of-page scroll when content count is divisible by 10 |

## DOM Stability

Each action ends with `WaitForDOMStable` -- a `MutationObserver` that resolves after `quiet_ms` milliseconds of DOM silence. This means the next tree tick starts from a fully-settled page, not from a mid-render state.

`WaitForDOMStable` uses `MutationObserver` so it responds to actual DOM activity. Unlike a fixed `asyncio.sleep`, it adapts to fast and slow renders equally.

## Using the Browser Fetchers

Browser-backed fetching is available through three fetcher types. Pass `--fetcher` on the CLI or `ScrapePolicy(fetcher_type=...)` in Python:

```bash
uv run yosoi --url https://example.com --fetcher waterfall
uv run yosoi --url https://example.com --fetcher headless
uv run yosoi --url https://example.com --fetcher headful
```

```python
policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(scrape=ys.ScrapePolicy(fetcher_type='waterfall')),
)
rows = await ys.scrape(url, Contract, policy=policy)
```

| Fetcher | Description |
|---------|-------------|
| `simple` | Plain HTTP -- fast, no browser, works for static HTML |
| `waterfall` | Simple → Headless → Headful (tries each tier in order) |
| `headless` | Headless Chrome via VoidCrawl |
| `headful` | Visible Chrome via VoidCrawl (best bot evasion) |

`waterfall` is the recommended choice for mixed workloads: it uses simple HTTP for static pages and escalates to Chrome only when needed.

## The Waterfall Fetcher

`JSFetcher` (the waterfall) runs three tiers in order and stops at the first that succeeds:

```
1. SimpleFetcher (plain HTTP)
   ├── success, no JS → return
   └── fail or requires_js → continue

2. HeadlessFetcher (headless Chrome + DOMLoader)
   ├── success → return
   └── fail → continue

3. HeadfulFetcher (visible Chrome + DOMLoader)
   └── return regardless (best-effort)
```

The winning tier for each domain is cached in `.yosoi/fetch/`. Subsequent runs skip the waterfall entirely and jump straight to the cached tier.

```python
policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    ys.Policy(scrape=ys.ScrapePolicy(fetcher_type='waterfall')),
)

for item in await ys.scrape('https://finance.yahoo.com/news', Product, policy=policy):
    print(item.get('headline'))
```

## Tuning DOMLoader

`DOMLoader` exposes several parameters through `HeadlessFetcher` and `HeadfulFetcher`:

| Parameter | Default | Effect |
|-----------|---------|--------|
| `max_cycles` | 20 | Maximum behavior tree restarts before stopping |
| `quiet_ms` | 800 | Milliseconds of DOM silence that counts as stable |
| `max_click_cycles` | 50 | Maximum clicks per trigger before exhaustion |
| `max_scroll_cycles` | 10 | Maximum scroll iterations for infinite scroll |

For most sites the defaults work well. Raise `max_scroll_cycles` for feeds with many pages; lower `quiet_ms` for fast-rendering SPAs.

## Requirements

Browser fetchers require VoidCrawl:

```bash
uv add voidcrawl
```

VoidCrawl is a Rust-native Chrome DevTools Protocol client exposed to Python via PyO3. See [VoidCrawl](/yosoi/voidcrawl/) for more detail.

## FAQs

<details>
<summary>When should I use the waterfall instead of the simple fetcher?</summary>

Use `waterfall` when you're scraping a mix of static and dynamic pages and don't want to think about which is which. It adds latency on static pages (one failed Chrome attempt before returning the plain HTTP result) but is otherwise transparent. For known dynamic sites, use `headless` or `headful` directly.

</details>

<details>
<summary>My page loads content but DOMLoader doesn't find it. What's wrong?</summary>

Run with `--debug` to save the HTML that Yosoi sees after DOMLoader finishes. If the content is present, the issue is with selector discovery -- not loading. If the content is absent, the page likely uses a trigger pattern not covered by the current catalogues (`catalogues.py`). Check which patterns DOMLoader probes for and compare against what the page actually uses.

</details>

<details>
<summary>Can I use DOMLoader without the full waterfall?</summary>

Yes -- `ScrapePolicy(fetcher_type='headless')` or `ScrapePolicy(fetcher_type='headful')` use DOMLoader directly without the Simple HTTP tier.

</details>

<details>
<summary>Does DOMLoader interact with authenticated pages?</summary>

Not currently. DOMLoader probes for publicly-observable DOM patterns (cookie banners, load-more buttons). It does not handle login forms, session cookies, or authentication flows. Pass pre-authenticated HTML to Yosoi if the target requires login.

</details>

## References

<a id="ref-1"></a>△ **VoidCrawl**. Cascading Labs. *Rust-native CDP browser automation for Python via PyO3.* https://github.com/CascadingLabs/VoidCrawl

<a id="ref-2"></a>○ **MutationObserver**. MDN Web Docs. *Web API for watching for changes to the DOM tree.* https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver

<a id="ref-3"></a>◑ **Behavior Trees**. Wikipedia. *Computational model used in robotics and game AI for task selection.* https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control)

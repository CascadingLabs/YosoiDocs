---
title: Fetchers
description: Choose how Yosoi retrieves HTML — plain HTTP, headless Chrome, or an adaptive waterfall.
---

Yosoi fetches HTML before selector discovery and extraction. Every fetch goes through an `HTMLFetcher` instance. The default is a fast plain-HTTP client; for JavaScript-heavy pages you can escalate to a browser-backed fetcher or use the adaptive waterfall that handles both.

## Fetcher Types

| Fetcher | CLI flag | Python value | Description |
|---------|----------|--------------|-------------|
| Simple | `--fetcher simple` | `'simple'` | Plain HTTP with realistic headers. Fast, no Chrome dependency. |
| Waterfall | `--fetcher waterfall` | `'waterfall'` | Simple → Headless → Headful. Adapts to the page automatically. |
| Headless | `--fetcher headless` | `'headless'` | Headless Chrome via VoidCrawl. |
| Headful | `--fetcher headful` | `'headful'` | Visible Chrome via VoidCrawl. Best bot evasion. |

### Simple Fetcher (default)

Sends HTTP requests with randomized user-agent headers and realistic browser fingerprints. Works for the majority of sites — any page where the content is present in the server response without needing JavaScript to render.

```bash
uv run yosoi --url https://qscrape.dev/l1/news --contract NewsArticle
```

```python
async for item in pipeline.scrape(url):  # fetcher_type defaults to 'simple'
    ...
```

When to use: static sites, news portals, most product catalogues, anything where `Cmd+U` / `Ctrl+U` in a browser shows the content you want to extract.

### Waterfall Fetcher

Tries three tiers in order and stops at the first that succeeds:

```
1. SimpleFetcher (plain HTTP)
   ├── success, no JS detected → return immediately
   └── failure or JS-rendered page → continue

2. HeadlessFetcher (headless Chrome + DOMLoader)
   ├── success → return immediately
   └── failure → continue

3. HeadfulFetcher (visible Chrome + DOMLoader)
   └── return result regardless (best-effort final tier)
```

The winning tier for each domain is cached in `.yosoi/fetch/`. On the next run, the waterfall is skipped entirely — Yosoi jumps straight to the cached tier.

```bash
uv run yosoi --url https://finance.yahoo.com --contract NewsArticle --fetcher waterfall
```

```python
async for item in pipeline.scrape(url, fetcher_type='waterfall'):
    ...
```

When to use: mixed workloads, sites you haven't tested yet, or anywhere you want Yosoi to adapt without configuring which tier to use.

### Headless and Headful Fetchers

Both tiers use VoidCrawl<sup>[△](#ref-1)</sup> to drive a Chrome instance and `DOMLoader` to bring the page to a fully-loaded state. The difference is visibility:

- **Headless**: Chrome runs without a window. Faster, suitable for most dynamic sites.
- **Headful**: Chrome shows a visible window. Harder for anti-bot systems to distinguish from a real user. Use when headless gets blocked.

Both require VoidCrawl to be installed:

```bash
uv add voidcrawl
```

See [DOMLoader](/guides/dom-loader/) for how the page-loading behavior tree works.

## Detecting Which Tier Is Needed

You don't need to figure this out yourself when using the waterfall — it runs a HEAD probe before the first full fetch to check for JavaScript signals:

- Content-Length under 5,000 bytes on an HTML response
- Framework headers (`X-Powered-By: Next.js`, server headers for Vercel/Netlify)
- Hard-block status codes (403, 429, 503)
- Chunked transfer with no Content-Length (streaming SSR)

When the HEAD probe returns positive, the waterfall skips Simple HTTP and starts from headless Chrome.

For a deeper explanation of what makes a site require JavaScript, see [Understanding the Web](/guides/understanding-the-web/).

## Strategy Cache

The waterfall stores the winning fetcher tier per domain in `.yosoi/fetch/`. This prevents re-running the three-tier probe on every request for a domain Yosoi has already learned about.

```
.yosoi/
  fetch/
    fetch_finance_yahoo_com.json
    fetch_shop_example_com.json
```

Each file records the tier and the highest selector strategy level that worked:

```json
{
  "domain": "finance.yahoo.com",
  "fetcher": "headless",
  "selector_level": "css",
  "discovered_at": "2026-05-23T14:00:00Z"
}
```

To re-run the waterfall for a specific domain (e.g. after a site redesign), delete its fetch file:

```bash
rm .yosoi/fetch/fetch_finance_yahoo_com.json
```

Or pass `--force` to also re-run selector discovery:

```bash
uv run yosoi --url https://finance.yahoo.com --contract NewsArticle --fetcher waterfall --force
```

## Using a Shared Fetcher Across URLs

When processing multiple URLs sequentially, passing a single fetcher instance avoids the overhead of creating and closing a client per URL. The waterfall's Chrome instances start lazily on first need and stay open for the batch.

```python
from yosoi.core.fetcher import create_fetcher

async with create_fetcher('waterfall') as fetcher:
    for url in urls:
        await pipeline.process_url(url, fetcher=fetcher)
```

`process_urls()` already does this internally — it creates one shared fetcher for the entire batch.

## FAQs

<details>
<summary>How do I know which fetcher a URL actually needed?</summary>

Check `.yosoi/fetch/` after the waterfall runs for a domain. The JSON file records which tier won. Run with `--debug` to also save the HTML Yosoi received from that tier.

</details>

<details>
<summary>Does concurrent processing (workers > 1) work with browser fetchers?</summary>

Yes. Each concurrent worker creates its own fetcher instance. For the waterfall, Chrome starts lazily per worker on first need. The per-domain strategy cache is shared across workers via the filesystem.

</details>

<details>
<summary>Can I pass custom Chrome arguments to the browser fetchers?</summary>

Not directly through the Pipeline API today. The VoidCrawl `BrowserConfig` is constructed inside the fetcher with `headless`, `stealth`, and `no_sandbox` options. Pass `no_sandbox=True` when running inside Docker or other sandboxed environments.

</details>

## References

<a id="ref-1"></a>△ **VoidCrawl**. Cascading Labs. *Rust-native CDP browser automation for Python via PyO3.* https://github.com/CascadingLabs/VoidCrawl

<a id="ref-2"></a>○ **DOMLoader**. Cascading Labs. *Behavior-tree page loader for Yosoi browser fetchers.* /guides/dom-loader/

<a id="ref-3"></a>◑ **Understanding the Web**. Cascading Labs. *How HTML, the DOM, and JavaScript frameworks affect what Yosoi can see.* /guides/understanding-the-web/

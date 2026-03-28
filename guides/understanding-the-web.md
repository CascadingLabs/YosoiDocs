---
title: Understanding the Web
description: How HTML, the DOM, and JavaScript frameworks affect what Yosoi can see and extract.
---

Not all websites are equally easy to scrape. Understanding why helps you choose the right approach and set realistic expectations before discovery runs.

## HTML vs. the DOM

When you load a page in a browser, the browser parses the raw HTML and constructs a live tree of objects called the DOM (Document Object Model). JavaScript can then modify that tree: adding nodes, removing them, rewriting text content, injecting entire components. What you see on screen may look nothing like the original HTML the server sent.

Yosoi fetches the raw HTML. It sees what the server returns, not what JavaScript builds after the fact. For most news sites, blogs, and product catalogues this is fine. The content is in the HTML. For single-page apps that load data after the initial render, Yosoi will get a mostly-empty shell.

If you can view the page source in your browser (`Cmd+U` / `Ctrl+U`) and see the text you want to extract, Yosoi can work with it. If the content only appears after JavaScript runs, you have two options:

- **DOM fetcher (beta):** Yosoi drives a real browser internally and passes the rendered DOM into the pipeline. See [A3Nodes](/guides/a3nodes/) for current status.
- **Manual pre-rendering:** Render the page yourself with Playwright or Puppeteer and pass the HTML to Yosoi directly. This is the stable path for now.

## Difficulty levels

### L1: Static HTML, CSS, and JS

Standard HTML pages with no frontend framework and no dynamic data loading. The full content is present in the server response. Selectors are stable and discovery is reliable.

These are the easiest targets. [QScrape L1 sites](https://qscrape.dev) simulate this category with realistic content and layouts.

### L2: SPAs and data-driven dynamic sites

Single-page applications and sites that load content after the initial HTML render. The server sends a minimal shell; JavaScript fetches data and injects it into the DOM. By the time the page looks right in a browser, the raw HTML Yosoi fetched may be largely empty.

For L2 sites, Yosoi's DOM fetcher (currently in beta) can handle rendering natively. Alternatively, render the page yourself with a headless browser (Playwright, Puppeteer) and pass the resulting HTML to Yosoi manually. Discovery then works against the rendered output either way.

[QScrape L2 sites](https://qscrape.dev) cover this category with Lit, Svelte, React, and Vue.

### L3: Anti-bot sites

Sites that actively try to detect and block automated access. Techniques include rate limiting, fingerprinting request headers, requiring session cookies, obfuscating markup, and serving different content to suspected bots.

Yosoi does not attempt to bypass anti-bot measures. If a site blocks the HTTP request, discovery fails. You are responsible for handling authentication, headers, and session management before passing HTML to Yosoi.

[QScrape L3 sites](https://qscrape.dev) apply generic anti-bot patterns (excluding captchas) for testing.

### L4: Captchas and traffic monitoring

The hardest class of sites. In addition to L3 protections, these deploy active challenge puzzles (CAPTCHAs, proof-of-work) and monitor traffic patterns across sessions to flag and block suspicious behavior.

Bypassing L4 protections is outside Yosoi's scope. Anti-captcha APIs and residential proxy networks exist for this purpose but are not integrated. QScrape does not include L4 sites.

## QScrape

[QScrape](https://qscrape.dev) is a purpose-built evaluation suite of fictional websites across the three difficulty levels above. Use it to:

- Validate that Yosoi's selector discovery works correctly for a given site type
- Regression-test your pipelines against a stable, unchanging source
- Benchmark discovery accuracy across providers

Because QScrape sites are controlled and stable, they are ideal for CI. Point Yosoi at a QScrape URL and assert on the extracted field values.

## FAQs

<details>
<summary>Can Yosoi scrape single-page apps (L2 sites)?</summary>

Yes, with the DOM fetcher (currently in beta). Yosoi can drive a real browser internally to render the page before running discovery and extraction. The stable alternative is to render yourself with a headless browser (Playwright, Puppeteer) and pass the HTML to Yosoi manually. See [A3Nodes](/guides/a3nodes/) for beta status.

</details>

<details>
<summary>Why do class names change between deploys on framework-based sites?</summary>

Build tools like webpack and Vite generate hashed class names to enable long-term caching. A deploy changes the hash, which invalidates any selector that targets that class. Yosoi's LLM-based discovery tries to avoid these by preferring stable attributes (IDs, data attributes, semantic tags), but it is not guaranteed.

</details>

<details>
<summary>What is the practical difference between L2 and L3 difficulty?</summary>

L2 adds frontend framework complexity but does not try to block you. L3 actively resists scraping. A tool that works on L2 will not necessarily work on L3.

</details>

<details>
<summary>Should I test against QScrape sites before hitting my real targets?</summary>

Yes. It is a fast way to verify your pipeline is wired up correctly before sending requests to sites you do not control.

</details>

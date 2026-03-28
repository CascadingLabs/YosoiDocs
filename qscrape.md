---
title: QScrape
description: A web scraper evaluation suite for benchmarking and testing scrapers against fictional test websites.
---

[QScrape](https://qscrape.dev) is a web scraper evaluation suite that provides a collection of fictional test websites across multiple difficulty levels, designed for benchmarking and testing the capabilities of web scrapers.

Built as a side project for Yosoi, QScrape gives you realistic targets to validate your scraping pipelines against, without hitting real sites.

- [qscrape.dev](https://qscrape.dev)
- [GitHub](https://github.com/CascadingLabs/QScrape)

## Difficulty Levels

### L1: Static HTML/CSS/JS

Standard sites with no dynamic data and no web frameworks. Currently available:

- **Mountainhome Herald**: a news portal
- **ScoreTap**: esports scores
- **Eldoria Registry of Deeds**: tax records
- **VaultMart**: e-commerce catalogue

### L2: Modern Frameworks

Sites built with Lit, Svelte, React, and Vue, covering typical DOM-level complexity found in the wild.

### L3: Anti-Bot

Sites with anti-bot protections (excluding captchas or challenge puzzles that would require anti-captcha APIs).

## Using QScrape with Yosoi

QScrape sites are ideal targets for testing Yosoi's selector discovery. Point Yosoi at any QScrape site to validate that discovered selectors correctly extract the expected content from a known, stable source.

## FAQs

<details>
<summary>Are QScrape sites suitable for production scraping?</summary>

No. They are fictional test targets designed for evaluation and benchmarking, not for collecting real data.

</details>

<details>
<summary>Do QScrape sites ever change their structure?</summary>

Occasionally, when new difficulty variants or layout scenarios are added. Check the [GitHub changelog](https://github.com/CascadingLabs/QScrape) before relying on a specific selector set in CI.

</details>

<details>
<summary>Can I add QScrape to my automated test suite?</summary>

Yes. Because the sites are stable and purpose-built, they work well as regression targets. Run Yosoi against a QScrape URL in your tests and assert on the extracted field values.

</details>

<details>
<summary>What is the difference between L2 and L3 sites?</summary>

L2 sites add framework complexity (React, Svelte, etc.) but make no attempt to block scrapers. L3 sites actively apply anti-bot techniques such as rate limiting or obfuscated markup.

</details>

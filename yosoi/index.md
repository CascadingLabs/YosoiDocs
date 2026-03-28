---
title: Introduction
description: AI-powered CSS selector discovery for web scraping.
---

Most web scraping tools make you choose between cost and reliability. LLM-per-request APIs are accurate but expensive at scale. Regex heuristics are cheap but brittle. Yosoi takes a different approach: use an LLM once per domain to discover selectors, cache the result, and scrape indefinitely at cost.

The thesis is simple. Selector discovery is a one-time cost. Extraction is the cost of your electricity bill.

## Technical stack

**Pydantic** is the foundation for contract definitions and data validation. You define what you want to extract as a typed Pydantic model. Yosoi handles the rest.

**Async native.** The scraping pipeline is built on `asyncio` from the ground up. `scrape()` is an async generator; `process_urls()` is fully non-blocking.

**Concurrent background workers.** `Pipeline.process_urls()` distributes work across a configurable worker pool. Discovery and extraction both run in parallel. A Rich Live progress table updates automatically when workers > 1.

## How it works

1. You provide a URL and a data contract (a typed Pydantic model describing what to extract)
2. Yosoi fetches the page HTML and sends it to an LLM
3. The LLM identifies stable CSS selectors for each field in the contract
4. Selectors are validated and stored in `.yosoi/selectors/`
5. Future scrapes use the cached selectors directly, with no LLM call needed

## FAQs

<details>
<summary>What is Yosoi?</summary>

A way to make web scraping cheaper with AI without sacrificing accuracy or speed. Instead of calling an LLM on every scrape or relying on flaky regex heuristics, Yosoi transpiles selectors automatically the first time, so you only pay once and reproduce results for free from then on.

</details>

<details>
<summary>How does Yosoi work?</summary>

You point it at a URL. Yosoi fetches the HTML, sends it to an LLM to identify stable CSS selectors, validates them, and caches the result. Every subsequent scrape of that domain uses the cached selectors directly with no LLM involved.

</details>

<details>
<summary>Why was Yosoi built?</summary>

For internal use at Cascading Labs. Every other API or LLM-per-request approach would have cost hundreds of thousands of dollars at scale. Yosoi was the only economically viable path, and we figured others had the same problem.

</details>

<details>
<summary>Does Yosoi work with JavaScript-rendered sites?</summary>

Yosoi fetches static HTML. For sites that require JavaScript execution to render content, pair it with a headless browser and pass the rendered HTML manually.

</details>

<details>
<summary>What happens if a site redesigns its layout?</summary>

The cached selectors will likely break. Delete the relevant file from `.yosoi/selectors/` and re-run discovery to generate fresh selectors for the updated layout.

</details>

<details>
<summary>Which LLM provider gives the best results?</summary>

Results are generally consistent across providers. Groq and Gemini are fast and cheap; OpenAI tends to be more conservative with selector choices. Run discovery with `--provider` to compare.

</details>

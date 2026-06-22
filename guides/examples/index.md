---
title: Examples
description: Working Yosoi examples using QScrape test sites.
slug: guides/examples
faqs:
  - q: "Why use QScrape instead of real sites in examples?"
    a: "Real sites change. A working example today might fail tomorrow if the target redesigns. QScrape sites are stable, controlled, and designed exactly for this purpose. They also avoid sending unwanted traffic to sites that did not ask for it."
  - q: "Can I run these examples against real sites?"
    a: "Yes. Swap any QScrape URL for a real one. Yosoi works the same way. The QScrape URLs are used here purely for documentation stability."
  - q: "Do I need a QScrape account?"
    a: "No. QScrape sites are publicly accessible with no authentication required."
---

All examples in this documentation use [QScrape](https://qscrape.dev)<sup>[△](#ref-1)</sup> sites rather than real public websites. QScrape provides stable, purpose-built scraping targets that won't break, won't rate-limit you, and won't be affected by the traffic this documentation generates.

If you want to follow along locally, every example here will work against the corresponding QScrape URL without modification.

Prefer the Python examples that call `ys.scrape(...)` and render with `ys.show(...)`. The docs keep selectors out of the main path so Yosoi can learn them from the contract and the page structure.

## Crawl index example

Use `crawl_index` when you want to discover links before extracting records. This seeds from the QScrape L1 entrypoint and lets the crawler decide what it sees.

```python
import asyncio
import yosoi as ys

SEEDS = ('https://qscrape.dev/l1/',)

async def main() -> None:
    policy = ys.Policy.for_crawl(
        'crawl.conservative',
        budget=ys.CrawlBudget(max_pages=4, max_depth=1, max_pages_per_host=4),
        scheduler=ys.SchedulerPolicy(max_workers=2, per_host_concurrency=1, politeness_delay=0),
        safety=ys.CrawlSafety(
            respect_robots=False,
            allow_redirects=False,
            allowed_hosts=('qscrape.dev',),
        ),
        fetcher_type='simple',
    )

    summary = await ys.crawl_index(SEEDS, policy=policy)
    ys.show(summary)

asyncio.run(main())
```

`crawl.conservative` is a bounded, low-concurrency preset. The explicit `CrawlSafety` keeps the crawl host-scoped, disables redirects, and opts out of robots only because QScrape is the maintained Yosoi demo target.

The Yahoo Finance reuse test is the exception: it is a live operator smoke for page identity reuse. Use it when you need to validate real rendered pages across same-shaped ticker URLs, not as a required CI fixture.

## FAQs

<details>
<summary>Why use QScrape instead of real sites in examples?</summary>

Real sites change. A working example today might fail tomorrow if the target redesigns. QScrape sites are stable, controlled, and designed exactly for this purpose. They also avoid sending unwanted traffic to sites that did not ask for it.

</details>

<details>
<summary>Can I run these examples against real sites?</summary>

Yes. Swap any QScrape URL for a real one. Yosoi works the same way. The QScrape URLs are used here purely for documentation stability.

</details>

<details>
<summary>Do I need a QScrape account?</summary>

No. QScrape sites are publicly accessible with no authentication required.

</details>

## References

<a id="ref-1"></a>△ **QScrape**. Cascading Labs. *Purpose-built fictional scraping targets for benchmarking and testing.* https://qscrape.dev

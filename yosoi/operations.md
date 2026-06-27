---
title: Operations API
description: Use Yosoi's canonical scrape, search, crawl, and map operation surfaces from Python or the CLI.
---

The current public surface is operation-first. Python helpers and CLI commands compile user input into typed request models, execute the same runtime, and return typed result envelopes that are safe to serialize.

| Operation | Python helper | Request | Result | CLI |
|---|---|---|---|---|
| Scrape URLs with contracts | `ys.scrape(...)` | `ys.ScrapeRequest` | `ys.ScrapeResult` | `yosoi scrape` |
| Search the web for source URLs | `ys.search(...)` | `ys.SearchRequest` | `ys.SearchResult` | `yosoi search` |
| Crawl seed URLs | `ys.crawl(...)` or `ys.run_crawl(...)` | `ys.CrawlRequest` | `ys.CrawlResult` or `CrawlRunSummary` | `yosoi crawl` |
| Map sitemap URLs or subdomains | `ys.map(...)` | `ys.MapRequest` | `ys.MapResult` | `yosoi map` |

## Scrape

`ys.scrape(...)` accepts one URL or many URLs and one contract or many contracts. It returns a `ScrapeResult` envelope with one `ScrapeUnitResult` per URL and contract pair.

```python
import yosoi as ys

result = await ys.scrape(
    'https://qscrape.dev/l1/news/articles/',
    ys.NewsArticle,
    model='groq:llama-3.3-70b-versatile',
)

for unit in result.results:
    print(unit.url, unit.contract, unit.status, unit.record_count)
    print(unit.records)
```

Stable fields on each unit include `selector_source`, `cache_decision`, `llm_used`, `llm_reason`, `quality_status`, `quality_issues`, `expected_record_count`, `record_count`, `records`, and `error`.

CLI equivalent:

```bash
uvx yosoi scrape https://qscrape.dev/l1/news/articles/ \
  --contract @NewsArticle \
  --model groq:llama-3.3-70b-versatile \
  --json
```

Use `--request request.json` when an agent or another service already produced a `ScrapeRequest` JSON document. Use `--dump-request` to inspect what the CLI will execute.

## Search

`ys.search(...)` wraps DDGS search and normalizes provider rows into ranked hits plus a URL list. Policy can provide durable defaults for backend, region, safesearch, page, time limit, and result count.

```python
import yosoi as ys

result = await ys.search(
    'Cascading Labs Yosoi selector discovery',
    limit=5,
    backend='google,bing,brave',
)

for hit in result.hits:
    print(hit.rank, hit.title, hit.url)
```

CLI equivalent:

```bash
uvx yosoi search "Cascading Labs Yosoi selector discovery" \
  --limit 5 \
  --backend google,bing,brave \
  --json
```

## Crawl

`ys.crawl(...)` runs the crawl coordinator over one or more seeds. For machine-safe output, build a `CrawlRequest` and call `ys.run_crawl(...)`.

```python
import yosoi as ys

request = ys.CrawlRequest.from_axes(
    ['https://qscrape.dev/'],
    limit=25,
    compact=True,
    policy=ys.Policy(crawl=ys.CrawlPolicy(budget=ys.CrawlBudget(max_pages=25))),
)
result = await ys.run_crawl(request)
print(result.status)
print(result.summary)
```

CLI equivalent:

```bash
uvx yosoi crawl https://qscrape.dev/ \
  --limit 25 \
  --compact \
  --json
```

Use `--stress --run-id ID` to store crawl-run metrics and emit compact, stress-friendly output.

## Map

`ys.map(...)` discovers sitemap URLs from `robots.txt`, default sitemap locations, and nested sitemap indexes. With subdomain discovery enabled, it shells out to `subfinder` and returns normalized host inventory.

```python
import yosoi as ys

site = await ys.map('qscrape.dev', max_urls=200)
print(site.status, site.root_host)
for row in site.urls[:10]:
    print(row.url)
```

CLI equivalent:

```bash
uvx yosoi map qscrape.dev --max-urls 200 --json
uvx yosoi map qscrape.dev --subdomains --json
```

`ys.MapResult` includes `sitemaps`, `urls`, `hosts`, `subdomains`, and `errors`, so it is a good source-discovery step before crawl or scrape planning.

---
title: Crawl Policy
description: Bound crawl depth, concurrency, robots behavior, host safety, and crawl-adjacent escalation.
---

`crawl` is the policy for bounded site traversal.

```yaml
crawl:
  mode: contract_focus
  budget:
    max_pages: 25
    max_depth: 2
  scheduler:
    max_workers: 3
    per_host_concurrency: 1
    politeness_delay: 1.0
  safety:
    respect_robots: true
    allow_redirects: false
    allow_cross_domain: false
```

## Top-level fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `mode` | enum | `contract_focus` | Crawl mode: `seed_hunt`, `contract_focus`, `structure_guarded`, or `explorer`. |
| `budget` | object | see below | Page, depth, attempt, and session limits. |
| `scheduler` | object | see below | Worker, per-host, politeness, timeout, and retry settings. |
| `safety` | object | see below | Robots, redirects, host allow/deny, path blocking. |
| `escalation` | object | default-deny | Paid/model escalation gates. |
| `target_contracts` | list | `[]` | Contracts the crawl is trying to find. |
| `scrape_contracts` | boolean or list | `false` | Whether to scrape representative URLs after crawl. |
| `scrape_url_limit_per_contract` | integer | `1` | Representative scrape cap. |
| `fetcher_type` | enum | `auto` | Fetch tier for crawl page acquisition. |

## Budget

| Field | Default | Description |
|---|---:|---|
| `max_pages` | `1` | Maximum successful pages. |
| `max_depth` | `0` | Link depth from seeds. |
| `max_attempts` | `null` | Optional cap including failures and policy-blocked URLs. |
| `max_pages_per_host` | `null` | Optional host-level page cap. |
| `crawl_session_id` | `null` | Checkpoint/session id when persistence is enabled. |

### The `limit` argument

`ys.crawl(seeds, limit=N)` and `yosoi crawl --limit N` are shorthand for the page budget: `limit` lowers the resolved `max_pages` to `N`, so the crawl fetches at most `N` pages.

`limit` also raises a preset's `max_pages_per_host` to `N` when the preset set it lower, so a per-host cap cannot silently answer `limit=40` with 30 pages. Politeness controls (`politeness_delay`, `per_host_concurrency`) are never changed by `limit`.

It is an upper bound, not a target. Depth, host safety rules, fetch failures, and plain reachability can all stop a crawl below `N`.

```python
summary = await ys.crawl('https://example.com/', limit=25)
assert summary.pages_fetched <= 25
```

For anything beyond a page cap -- depth, attempts, per-host allocation, session persistence -- set `budget` on the policy directly.

## Scheduler

| Field | Default | Description |
|---|---:|---|
| `max_workers` | `1` | Total concurrent crawl workers. |
| `per_host_concurrency` | `1` | Concurrent workers per host. |
| `politeness_delay` | `1.0` | Minimum delay between same-host fetches. |
| `fetch_timeout_seconds` | `15.0` | Fetch timeout for crawl pages. |
| `max_fetch_retries` | `2` | Fetch retry cap. |

## Safety

| Field | Default | Description |
|---|---:|---|
| `respect_robots` | `true` | Honor robots.txt. |
| `allow_redirects` | `false` | Allow redirects during crawl fetches. |
| `allow_cross_domain` | `false` | Permit crawling across domains. |
| `allowed_hosts` | `[]` | Explicit host allowlist. If empty, seeds derive allowed hosts. |
| `denied_hosts` | `[]` | Explicit host denylist. |
| `blocked_path_prefixes` | `[]` | Path prefixes to skip before fetch. |

## Escalation

Escalation is default-deny.

```yaml
crawl:
  escalation:
    allow_model_discovery: false
    allow_paid_scrapers: false
    max_llm_calls: 0
    max_paid_scraper_calls: 0
```

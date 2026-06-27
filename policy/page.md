---
title: Page Policy
description: Configure generic page acquisition shared by scrape, crawl, and scripts.
---

`page` controls how Yosoi acquires and cleans a page before the runtime decides what to do with it.

```yaml
page:
  fetcher_type: auto
  timeout_seconds: 30.0
  max_fetch_retries: 1
  allow_redirects: true
  clean_html: true
  cleaner_profile: discovery
  chrome_ws_urls: []
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `fetcher_type` | enum | `auto` | `auto`, `simple`, `headless`, `headful`, or `waterfall`. |
| `timeout_seconds` | number | `30.0` | Page fetch timeout. |
| `max_fetch_retries` | integer | `1` | Retry attempts for page acquisition. |
| `allow_redirects` | boolean | `true` | Whether generic page fetches may follow redirects. |
| `clean_html` | boolean | `true` | Clean noisy HTML before discovery/extraction. |
| `cleaner_profile` | enum | `discovery` | `discovery` or `raw`. |
| `chrome_ws_urls` | list[string] | `[]` | Existing Chrome WebSocket endpoints for browser-backed fetchers. |

`page` is shared. Scrape and crawl policies can still override page behavior for their specific runtime.

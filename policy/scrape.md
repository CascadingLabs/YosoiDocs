---
title: Scrape Policy
description: Configure replay, discovery, selector level, concurrency, and browser security posture for scrape runs.
---

`scrape` controls one scrape execution.

```yaml
scrape:
  force: false
  skip_verification: false
  fetcher_type: auto
  selector_level: 5
  max_concurrency: null
  cross_origin_dom: false
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `force` | boolean | `false` | Force rediscovery even if cached selectors exist. |
| `skip_verification` | boolean | `false` | Skip selector verification after extraction. Faster, less safe. |
| `fetcher_type` | enum | `auto` | `auto`, `simple`, `headless`, `headful`, or `waterfall`. |
| `selector_level` | integer | max level | Maximum selector strategy level to use. CLI `all` maps to the max level. |
| `max_concurrency` | integer or null | `null` | Optional URL worker cap. |
| `cross_origin_dom` | boolean | `false` | Disable browser site-isolation protections for cross-origin iframe access. Explicit opt-in only. |

## When to set force

Use `force: true` when the website changed and you want to replace cached selectors.

```yaml
scrape:
  force: true
```

## Cross-origin DOM warning

`cross_origin_dom: true` weakens the browser security posture for the session. Use it only for pages where Yosoi must inspect embedded cross-origin frames and you understand the risk.

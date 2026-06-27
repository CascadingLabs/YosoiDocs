---
title: Search Policy
description: Configure defaults for ys.search and yosoi search.
---

`SearchPolicy` controls durable defaults for web search. Direct Python arguments and CLI flags still win for the current run.

```python
import yosoi as ys

policy = ys.Policy(
    search=ys.SearchPolicy(
        provider='ddgs',
        backend='google,bing,brave',
        region='us-en',
        safesearch='moderate',
        max_results=10,
        page=1,
    )
)

result = await ys.search('selector discovery scraping', policy=policy)
print(result.urls)
```

YAML policy:

```yaml
search:
  kind: text
  provider: ddgs
  backend: google,bing,brave
  region: us-en
  safesearch: "moderate"
  max_results: 10
  page: 1
  timelimit: null
```

## Fields

| Field | Default | Description |
|---|---:|---|
| `kind` | `text` | Search kind. Text search is the current public mode. |
| `provider` | `ddgs` | Search provider. |
| `backend` | `google,bing,brave` | DDGS backend string. |
| `region` | `us-en` | Search region. |
| `safesearch` | `moderate` | One of `on`, `moderate`, or `off`. |
| `max_results` | `10` | Number of results to request. |
| `page` | `1` | Search result page. |
| `timelimit` | `null` | Optional DDGS time limit, such as `d`, `w`, `m`, `y`, or a provider-supported range. |

## Environment

`Policy.from_env()` reads:

- `YOSOI_SEARCH_BACKEND`
- `YOSOI_SEARCH_REGION`
- `YOSOI_SEARCH_SAFESEARCH`
- `YOSOI_SEARCH_MAX_RESULTS`
- `YOSOI_SEARCH_PAGE`
- `YOSOI_SEARCH_TIMELIMIT`

## CLI

```bash
uvx yosoi search "site:cascadinglabs.com yosoi" \
  --limit 5 \
  --backend google,bing,brave \
  --json
```

Use `--dump-request` to print the resolved `SearchRequest` JSON without executing the search.

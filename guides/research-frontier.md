---
title: Research Frontier Packets
description: Turn exploratory search, crawl, and scrape runs into local evidence packets before promoting a deterministic Yosoi pipeline.
---

Research frontier mode is for early source discovery and competitor or market research. It keeps exploratory work local, append-only, and auditable before you promote any contract or source into production scraping.

## Create a packet

```bash
uvx yosoi research init "AI selector discovery competitors"
```

This creates a timestamped packet under `.yosoi/research/` with:

- `frontier.json` metadata and budget mode
- `policy.yaml` copied from the zero-cost or budgeted template
- `query-plan.json`, `source-map.json`, `claims.json`
- `observations.jsonl` and `evidence.jsonl`
- folders for `sources/`, `candidate-contracts/`, `scrape-results/`, and `notes/`

Pass budgets when you explicitly allow paid LLM or API work:

```bash
uvx yosoi research init "AI selector discovery competitors" \
  --llm-budget-usd 5 \
  --api-budget-usd 10
```

## Add observations

Use normal operation commands to produce artifacts, then append observations to the packet.

```bash
uvx yosoi search "AI selector discovery web scraping" --json > .yosoi/research/RUN/sources/search-001.json
uvx yosoi research observe .yosoi/research/RUN search .yosoi/research/RUN/sources/search-001.json

uvx yosoi crawl https://example.com --limit 20 --json > .yosoi/research/RUN/sources/crawl-001.json
uvx yosoi research observe .yosoi/research/RUN crawl .yosoi/research/RUN/sources/crawl-001.json

uvx yosoi scrape https://example.com/article --contract @NewsArticle --json > .yosoi/research/RUN/scrape-results/scrape-001.json
uvx yosoi research observe .yosoi/research/RUN scrape .yosoi/research/RUN/scrape-results/scrape-001.json

uvx yosoi research observe .yosoi/research/RUN note "Source blocks pricing pages behind auth."
```

Scrape observations preserve contract fingerprints, selector source, cache decision, LLM use, quality status, quality issues, expected record count, and record count from the canonical `ScrapeResult` envelope.

## Check promotion status

```bash
uvx yosoi research status .yosoi/research/RUN
uvx yosoi research status .yosoi/research/RUN --json
```

The status view summarizes contract promotion states and open quality gaps. Use it to decide whether a candidate contract is still exploratory, provisional, validated, rejected, or production-ready.

## Promotion rule

Do not put research packet output directly on a hot path. Promote only after you have:

1. deterministic source URLs or a repeatable map or crawl step,
2. a stable contract with useful quality status,
3. known budget boundaries in `policy.yaml`, and
4. a production pipeline plan that removes exploratory LLM/API calls from the common path.

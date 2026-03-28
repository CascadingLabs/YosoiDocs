---
title: Roadmap
description: Planned features and future direction for Yosoi.
---

If something isn't shipped yet, you'll find it in one to three places:

1. **Here** -- on this page, as a planned item
2. **[GitHub Projects](https://github.com/orgs/CascadingLabs/projects)** -- for work that is actively planned or in progress
3. **[GitHub Issues](https://github.com/CascadingLabs/Yosoi/issues)** -- as a tracked issue in the Yosoi repo

If you want to follow progress on a specific feature, the GitHub issue is the most granular source. If you want a high-level picture of where the project is headed, start here.

---

## In Beta

### DOM-Enabled Scraping

Yosoi can now drive a real browser to fetch and render JavaScript-heavy pages without manual pre-rendering. L2 sites (SPAs, framework-rendered apps) no longer require you to run Playwright yourself before passing HTML in.

The DOM fetcher slots into the existing pipeline in place of the HTTP fetcher. Discovery and extraction code stays the same; only the HTML source changes from a raw server response to a rendered DOM snapshot.

Core functionality works. The public API is still subject to change, and authenticated sessions and smart auto-detection are not yet available.

See [A3Nodes](/guides/a3nodes/) for architecture details and known limitations.

---

## Proposed

### A3Nodes: Node-Based Pipeline Architecture

The internal pipeline is being refactored into discrete, composable async nodes. Each stage (fetching, cleaning, discovery, extraction, validation) becomes an independently swappable unit. The DOM fetcher is built on this foundation, and future fetcher types will be too.

Proposed node types beyond the DOM fetcher:
- Authenticated fetcher: handles session cookies and login flows
- Proxy rotation: routes requests through a proxy pool
- Custom wait strategy: configurable DOM settling conditions for slow SPAs

Design is in progress. See [GitHub Projects](https://github.com/orgs/CascadingLabs/projects) for active planning.

---

## Planned

### QScrape L2 Sites

Fictional SPA test sites (React, Vue, Svelte, Lit) for benchmarking the DOM fetcher against known, stable targets. Mirrors the existing L1 suite.

### Distributed Selector Cache

Redis-backed selector storage for teams sharing discovery results across machines and CI runs. See [Scaling](/guides/scaling/).

### Versioned Docs

Per-release documentation snapshots so older API versions remain accessible.

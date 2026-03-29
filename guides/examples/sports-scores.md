---
title: Sports Scores
description: Extract sports scores using Yosoi.
---

Target: [ScoreTap](https://qscrape.dev/l1/scoretap) (QScrape L1)

This example extracts match results from an esports scoreboard. The contract uses `ys.Field()` for domain-specific fields that don't map to a built-in type, and pins the repeating container with `root`.

## CLI

There is no built-in contract for sports scores, so save the contract to a file and point the CLI at it:

```bash
uv run yosoi --url https://qscrape.dev/l1/scoretap --contract scores.py:Match --output json
```

## Python

```python
# scores.py
import asyncio
import yosoi as ys

class Match(ys.Contract):
    root = ys.css('div.match-row')

    teams: str = ys.Field(description='Team names')
    score: str = ys.Field(description='Final score')
    date: str = ys.Datetime()

async def main():
    pipeline = ys.Pipeline(ys.auto_config(), contract=Match)

    async for item in pipeline.scrape('https://qscrape.dev/l1/scoretap'):
        print(item.get('teams'), item.get('score'))

asyncio.run(main())
```

Run it:

```bash
uv run python scores.py
```

## What to Expect

Each row in the scoreboard yields one item with team names, the final score, and a parsed date. The `description` on each `ys.Field()` tells the AI what to look for during selector discovery -- this is especially important for generic fields that don't have a built-in type to guide the LLM.

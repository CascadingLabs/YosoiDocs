---
title: Atom Reads & Trust
description: Control field-atom replay and provenance trust tiers.
---

Two top-level fields gate atom replay:

```yaml
atom_reads: false
trust_tier: strict
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `atom_reads` | boolean | `false` | Allow reads from the field-atom index on legacy selector-cache miss. |
| `trust_tier` | enum | `strict` | `strict` serves trusted sources only. `yellow` serves all tiers, including quarantined sources. |

## Strict vs yellow

`strict` is the default. It serves only verified/manual/LLM atoms and quarantines risky fingerprint-generalized reuse.

`yellow` means let it ride. It can serve every provenance tier, including quarantined sources. Use it for experiments, not default production policy.

## CLI example

```bash
uvx yosoi scrape https://example.com --policy 'atom_reads: true'
uvx yosoi scrape https://example.com --policy 'atom_reads: true
trust_tier: yellow'
```

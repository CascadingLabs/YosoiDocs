---
title: Page Identity and Reuse
description: Reuse discovered selectors across pages with the same shape while keeping trust and drift behavior explicit.
---

Page identity reuse lets Yosoi discover a selector once, save it as a field atom, and reuse it on other pages that share the same page shape. The current implementation is intentionally conservative: reuse is keyed by an exact page-shape bucket and guarded by a policy object. If the atom index cannot answer every requested field unambiguously, Yosoi falls back to normal discovery.

This is the first integration slice from the old generalization work. The fuzzy fingerprint scorer exists for measurement and diagnostics, but it is not the read key yet.

## What Gets Reused

Yosoi stores per-field atoms instead of whole selector-cache files:

| Unit | Purpose |
| --- | --- |
| `ContractSpec` | A serializable contract shape that can be fingerprinted, stored, or passed inline |
| `PageFingerprint` | A content-free page-shape signature for comparing HTML structure |
| `FieldAtom` | A reusable field selector keyed by page shape, region role, field name, and Yosoi type |
| `Policy` | The trust boundary for atom reads and source tiers |

An atom is domain-aware but not domain-owned. `domains_seen` is provenance; the read key is the page shape plus field identity.

## Policy Controls

Atom reads are off unless the policy enables them. The default trust tier is strict.

```bash
YOSOI_ATOM_READS=1
YOSOI_ATOM_TRUST=strict
```

Strict trust accepts selector sources that were verified, manual, or LLM-discovered. Yellow trust also allows fingerprint-tier reuse.

```python
from yosoi.policies import Policy

policy = Policy(atom_reads=True, trust_tier='strict')
```

The resolver accepts a policy value directly. Deeper resolver code should not read environment variables itself; environment parsing belongs at the boundary.

## Inline Contract Specs

`resolve_contract()` now accepts a registered contract name, a `ContractSpec`, or a dictionary that validates as a `ContractSpec`.

```python
import yosoi as ys
from yosoi.utils.contracts import resolve_contract

class Quote(ys.Contract):
    name: str = ys.Field(description='company name')
    price: str = ys.Field(description='current share price')

spec = Quote.to_spec()
resolved = resolve_contract(spec)
```

If an inline spec matches an existing contract fingerprint, Yosoi returns the registered contract. Otherwise, it builds a runtime contract from the spec.

## Expected Behavior

Use this checklist when testing page identity reuse:

1. Discover on one page with atom reads enabled.
2. Scrape another page with the same shape.
3. Confirm the second page can resolve from atoms without rediscovering selectors.
4. Change the target to a structurally different page and confirm Yosoi falls back to discovery.
5. Run with strict trust first; use yellow trust only for experiments where fingerprint-tier reuse is acceptable.

## Current Limits

- Fuzzy `PageFingerprint.similarity()` is not wired into reads.
- Reuse is exact-bucket only.
- There is no verify-on-reuse step yet.
- Drift health and signal-lane queues are tracked follow-up work.
- Network endpoint signatures need VoidCrawl endpoint capture before they can become part of the page identity model.

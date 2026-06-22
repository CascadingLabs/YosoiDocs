---
title: Page Identity and Reuse
description: Reuse discovered selectors across pages with the same shape while keeping trust and drift behavior explicit.
faqs:
  - q: "Is page identity reuse the same thing as fuzzy fingerprint matching?"
    a: "No. Page identity reuse is the selector-serving path. Today it uses exact page-shape buckets and policy checks. Fuzzy fingerprinting is useful diagnostic evidence, but it is not the read key."
  - q: "What happens when an atom read is ambiguous?"
    a: "Yosoi falls back to normal discovery. Atom reads must answer every requested field unambiguously before they can serve a scrape."
  - q: "Why are atoms field-level instead of contract-level?"
    a: "Field atoms let compatible contracts share individual selector facts without pretending the whole contract is the same. A contract with title and url can share those facts with a larger contract that also asks for snippet."
  - q: "Should I use yellow trust in production?"
    a: "Start with strict trust. Yellow trust is for experiments where serving quarantined fingerprint-tier evidence is acceptable and you are watching the output closely."
---

Page identity reuse lets Yosoi discover a selector once, save it as a field atom, and reuse it on other pages that share the same page shape. The current implementation is intentionally conservative: reuse is keyed by an exact page-shape bucket and guarded by a policy object. If the atom index cannot answer every requested field unambiguously, Yosoi falls back to normal discovery.

This is the first integration slice from the old generalization work. The fuzzy [fingerprinting](/guides/fingerprinting/) scorer exists for measurement and diagnostics, but it is not the read key yet.

<figure class="ys-figure">
  <div class="ys-flow">
    <div class="ys-node">
      <div class="ys-node__label">Discover</div>
      <div class="ys-node__title">Selectors verified on a page</div>
      <div class="ys-node__body">Normal discovery still owns the first selector facts for a contract and page.</div>
    </div>
    <div class="ys-node ys-node--accent">
      <div class="ys-node__label">Store</div>
      <div class="ys-node__title">Field atoms</div>
      <div class="ys-chip-grid">
        <span class="ys-chip">page_shape</span>
        <span class="ys-chip">region_role</span>
        <span class="ys-chip">field_name</span>
        <span class="ys-chip">yosoi_type</span>
      </div>
    </div>
    <div class="ys-node">
      <div class="ys-node__label">Gate</div>
      <div class="ys-node__title">Policy and trust</div>
      <div class="ys-node__body">Atom reads must be enabled, unambiguous, and allowed by the trust tier.</div>
    </div>
    <div class="ys-node">
      <div class="ys-node__label">Fallback</div>
      <div class="ys-node__title">Discovery remains the escape path</div>
      <div class="ys-node__body">Any ambiguity, missing field, or disallowed source falls back to normal discovery.</div>
    </div>
  </div>
  <figcaption>Page identity reuse is exact-bucket and policy-gated today. Fuzzy fingerprints are evidence, not the read key.</figcaption>
</figure>

## What Gets Reused

Yosoi stores per-field atoms instead of whole selector-cache files:

| Unit | Purpose |
| --- | --- |
| `ContractSpec` | A serializable contract shape that can be fingerprinted, stored, or passed inline |
| `PageFingerprint` | A content-free page-shape signature for comparing HTML structure |
| `FieldAtom` | A reusable field selector keyed by page shape, region role, field name, and Yosoi type |
| `Policy` | The trust boundary for atom reads and source tiers |

An atom is domain-aware but not domain-owned. `domains_seen` is provenance; the read key is the page shape plus field identity. For rootless fields, the region fallback includes the contract name, so those atoms are intentionally less general.

## Policy Controls

Atom reads are off unless the policy enables them. The default trust tier is strict.

```bash
YOSOI_ATOM_READS=1
YOSOI_ATOM_TRUST=strict
```

Strict trust accepts selector sources that were verified, manual, or LLM-discovered. `fingerprint` is classified as quarantined evidence: strict serving rejects it, while yellow trust may serve it as quarantined output.

```python
from yosoi.policy import Policy

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

## FAQs

<details>
<summary>Is page identity reuse the same thing as fuzzy fingerprint matching?</summary>

No. Page identity reuse is the selector-serving path. Today it uses exact page-shape buckets and policy checks. Fuzzy fingerprinting is useful diagnostic evidence, but it is not the read key.

</details>

<details>
<summary>What happens when an atom read is ambiguous?</summary>

Yosoi falls back to normal discovery. Atom reads must answer every requested field unambiguously before they can serve a scrape.

</details>

<details>
<summary>Why are atoms field-level instead of contract-level?</summary>

Field atoms let compatible contracts share individual selector facts without pretending the whole contract is the same. A contract with `title` and `url` can share those facts with a larger contract that also asks for `snippet`.

</details>

<details>
<summary>Should I use yellow trust in production?</summary>

Start with strict trust. Yellow trust is for experiments where serving quarantined fingerprint-tier evidence is acceptable and you are watching the output closely.

</details>

---
title: Policy Files
description: Write Yosoi policy as JSON or YAML, either as one file or a modular directory tree.
---

Policy files use the `ys.Policy` shape directly. Do not wrap them in `yosoi:` or `policy:`.

For editor completions and validation, put the YAML language-server directive at the top:

```yaml
# yaml-language-server: $schema=https://cascadinglabs.com/yosoi/schemas/policy.schema.json
atom_reads: false
trust_tier: strict
scrape:
  fetcher_type: auto
search:
  backend: google,bing,brave
  max_results: 10
output:
  formats: [jsonl]
```

## Supported formats

Yosoi loads these extensions:

- `.yaml`
- `.yml`
- `.json`

YAML is usually better for hand-authored policy. JSON is useful for generated policy and CI artifacts.

## File locations

Yosoi discovers policy in global and project scopes.

```text
~/.config/yosoi/policy.yaml
~/.config/yosoi/policy.yml
~/.config/yosoi/policy.json
~/.config/yosoi/policy/**/*.yaml
~/.config/yosoi/policy/**/*.yml
~/.config/yosoi/policy/**/*.json

./.yosoi/policy.yaml
./.yosoi/policy.yml
./.yosoi/policy.json
./.yosoi/policy/**/*.yaml
./.yosoi/policy/**/*.yml
./.yosoi/policy/**/*.json
./yosoi.policy.yaml
./yosoi.policy.yml
./yosoi.policy.json
```

The flat file is still supported. The `policy/` directory is for modular policy.

## Modular directory example

```text
.yosoi/
  policy.yaml              # broad project defaults
  policy/
    00-model.yaml
    10-output.yaml
    20-page.yaml
    crawl/
      00-budget.yaml
      10-safety.yaml
    local.dev.yaml
```

Yosoi walks `policy/` recursively, loads JSON/YAML files only, and cascades them in lexicographic relative-path order. Later files override earlier files.

## Inline policy

The CLI accepts inline YAML or JSON through `--policy`:

```bash
uvx yosoi policy effective --policy 'atom_reads: true' --format yaml
uvx yosoi scrape https://example.com --policy '{"page": {"fetcher_type": "simple"}}'
```

Use inline policy for experiments, not durable project decisions.

## Validation

```bash
uvx yosoi policy validate .yosoi/policy.yaml
uvx yosoi policy inspect .yosoi/policy.yaml --format yaml
uvx yosoi policy effective --format json
```

## Starter file and schema

Create a starter policy file with the schema directive already included:

```bash
uvx yosoi policy init --local
uvx yosoi policy init --global
uvx yosoi policy init --global --local
```

Print the JSON Schema used by editors:

```bash
uvx yosoi policy schema
```

The hosted schema URL is:

```text
https://cascadinglabs.com/yosoi/schemas/policy.schema.json
```

Invalid discovered policy fails fast. Yosoi does not silently skip a malformed YAML or JSON policy file.

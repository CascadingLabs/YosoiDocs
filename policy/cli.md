---
title: Policy CLI
description: Validate, inspect, resolve, and pass policy files through the Yosoi CLI.
---

The `yosoi policy` command group is the policy workbench.

## Init

Create a starter YAML policy with an editor schema directive:

```bash
uvx yosoi policy init --local
uvx yosoi policy init --global
uvx yosoi policy init --local --force
```

The generated file starts with:

```yaml
# yaml-language-server: $schema=https://cascadinglabs.com/yosoi/schemas/policy.schema.json
```

## Schema

Print the JSON Schema for editor/tooling integration:

```bash
uvx yosoi policy schema
```

The same schema is hosted at:

```text
https://cascadinglabs.com/yosoi/schemas/policy.schema.json
```

## Validate

```bash
uvx yosoi policy validate .yosoi/policy.yaml
uvx yosoi policy validate .yosoi/policy/test.yaml --json
```

Validation parses JSON/YAML and validates the result against `ys.Policy`.

## Inspect

Normalize a policy file or inline snippet:

```bash
uvx yosoi policy inspect .yosoi/policy.yaml
uvx yosoi policy inspect .yosoi/policy.yaml --format yaml
uvx yosoi policy inspect 'atom_reads: true' --format json
```

## Defaults

Print empty policy defaults, or crawl defaults:

```bash
uvx yosoi policy defaults
uvx yosoi policy defaults --crawl --format yaml
```

## Effective

Print the resolved env + global + project policy stack:

```bash
uvx yosoi policy effective --format yaml
```

Add explicit layers:

```bash
uvx yosoi policy effective \
  --policy .yosoi/policy/20-page.yaml \
  --policy 'output: {flat_files: true}' \
  --format yaml
```

Skip global/project discovery:

```bash
uvx yosoi policy effective --no-discover --policy .yosoi/policy/test.yaml
```

## Scrape, discover, and crawl

Most run commands accept `--policy`:

```bash
uvx yosoi scrape https://example.com --policy .yosoi/policy.yaml
uvx yosoi discover https://example.com --policy 'scrape: {fetcher_type: headless}'
uvx yosoi crawl https://example.com --policy .yosoi/policy/crawl/10-budget.yaml
```

`--policy` can be repeated. Later `--policy` arguments override earlier ones. Direct CLI flags still win over `--policy`.

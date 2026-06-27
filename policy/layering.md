---
title: Layering & Precedence
description: How Yosoi combines environment, global files, project files, inline policy, and CLI flags.
---

Yosoi policy is layered. Each layer is a partial `Policy`. Only fields explicitly set by a layer participate in the merge.

## CLI precedence

For CLI runs, the effective order is:

```text
env < global policy files < project policy files < --policy sources < CLI flags
```

Higher layers win.

## Python precedence

In Python, call `Policy.cascade(...)` yourself:

```python
policy = ys.Policy.cascade(
    ys.Policy.from_env(),
    team_policy,
    project_policy,
    contract_policy,
    call_site_policy,
)
```

The last policy wins when layers set the same field.

## Nested merge behavior

Nested policy objects field-merge. This means a project can set one output option while preserving another option from a lower layer.

```yaml
# global
output:
  formats: [json]
  flat_files: true
```

```yaml
# project
output:
  formats: [jsonl]
```

Effective result:

```yaml
output:
  formats: [jsonl]
  flat_files: true
```

## Model credential firewall

`model` has one important special rule. If a higher layer changes `provider` or `model_name`, it replaces the model identity so a lower layer's credentials cannot accidentally travel to a different provider.

Safe pattern:

```yaml
model:
  provider: groq
  model_name: llama-3.3-70b-versatile
  credential_ref:
    source: env
    name: GROQ_KEY
```

## Effective policy

Use this before running a scrape when you are not sure which layer is winning:

```bash
uvx yosoi policy effective --format yaml
```

To test only explicit sources and skip discovery:

```bash
uvx yosoi policy effective --no-discover --policy .yosoi/policy/test.yaml --format yaml
```

---
title: Discovery Policy
description: Configure selector discovery backend, concurrency, lesson cache, and replay verification threshold.
---

`discovery` controls selector discovery behavior.

```yaml
discovery:
  max_concurrent: 5
  mode: auto
  lesson_cache: true
  replay_verify_threshold: 1.0
  static_mode_warning: true
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `max_concurrent` | integer | `5` | Maximum concurrent discovery tasks. |
| `mode` | enum | `auto` | `auto`, `static`, or `mcp`. |
| `lesson_cache` | boolean | `true` | Use learned discovery lessons when available. |
| `replay_verify_threshold` | number | `1.0` | Required replay verification confidence, from `0.0` to `1.0`. |
| `static_mode_warning` | boolean | `true` | Warn when static mode may miss dynamic behavior. |

## Modes

| Mode | Meaning |
|---|---|
| `auto` | Let Yosoi choose the available discovery path. |
| `static` | Use static discovery behavior. Useful for deterministic constrained runs. |
| `mcp` | Use MCP-backed discovery when configured. |

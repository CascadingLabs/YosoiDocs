---
title: Output Policy
description: Control terminal output, persisted formats, debug HTML, logs, and flat files.
---

`output` controls human and file output.

```yaml
output:
  formats: [json]
  quiet: true
  json_output: false
  plain_output: false
  debug_html: false
  debug_html_dir: .yosoi/debug
  flat_files: false
  logs: true
```

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `formats` | list[string] | `[]` | Persisted output formats, such as `json`, `jsonl`, `csv`, `xlsx`, or `parquet`. |
| `quiet` | boolean | `true` | Minimize human terminal output. |
| `json_output` | boolean | `false` | Machine JSON output mode. |
| `plain_output` | boolean | `false` | Plain terminal output. |
| `debug_html` | boolean | `false` | Save extracted HTML/debug artifacts. |
| `debug_html_dir` | path | `.yosoi/debug` | Debug artifact directory. |
| `flat_files` | boolean | `false` | Mirror extracted content to `.yosoi/content`. |
| `logs` | boolean | `true` | Enable local run logs. |

For automation, prefer CLI `--json` or policy `json_output: true`. For audit artifacts, use `formats` and `flat_files`.

---
title: Download Policy
description: Default-deny controls for file download side effects.
---

`download` controls whether a scrape run may download files.

```yaml
download:
  allow: false
  allowed_types: []
  directory: null
  max_bytes: null
  keep: true
```

Downloads are disabled by default. Enabling downloads should be explicit in durable policy.

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `allow` | boolean | `false` | Allows downloads for the run. Required for any file download. |
| `allowed_types` | list[string] | `[]` | Run-level allowlist such as `pdf`, `csv`, or `json`. |
| `directory` | string or null | `null` | Download output directory. |
| `max_bytes` | integer or null | `null` | Run-level file size cap. |
| `keep` | boolean | `true` | Keep downloaded files after extraction. |

## Example

```yaml
download:
  allow: true
  allowed_types: [csv, pdf]
  directory: .yosoi/downloads
  max_bytes: 25000000
  keep: true
```

The contract field still needs its own `ys.File(...)` configuration. The run policy and field policy must both allow the download.

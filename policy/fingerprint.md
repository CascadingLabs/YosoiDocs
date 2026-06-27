---
title: Fingerprint Policy
description: Configure the off-path fingerprint signal lane.
---

`fingerprint` controls the background signal lane for page fingerprint and health gathering.

```yaml
fingerprint:
  signal_lane: true
  backpressure: defer
  max_queue: 256
```

Leaving `fingerprint` unset means no fingerprint lane is created.

## Fields

| Field | Type | Default | Description |
|---|---|---:|---|
| `signal_lane` | boolean | `true` | Enable the off-path signal lane when the sub-policy is attached. |
| `backpressure` | enum | `defer` | `defer` keeps work for later. `drop` bounds memory by dropping work. |
| `max_queue` | integer | `256` | Maximum queued signal jobs. |

Fingerprint gathering is separate from trust decisions. The trust gate lives in top-level `trust_tier` and atom/source policy.

# Temporary Blocking

This page explains Block's exponentially growing duration, how repeated blocks accumulate on the record, and the risk when `BlockTimeMin` / `BlockTimeMax` are 0.

## Duration

```go
if err := sentry.Manager.Block.Add("192.0.2.33", "manual review"); err != nil {
	log.Printf("block: %v", err)
}
```

| Call | Record `count` | Duration |
|---|---|---|
| 1st (not currently blocked) | 1 | `BlockTimeMin` |
| nth (still blocked, n ≥ 2) | n | `2^n × BlockTimeMin`, capped at `BlockTimeMax` |

With `BlockTimeMin = 5m` and `BlockTimeMax = 24h` (the first 4 rows are measured):

| Call | Redis TTL |
|---|---|
| 1 | 5m |
| 2 | 20m |
| 3 | 40m |
| 4 | 1h20m |
| 9 | 24h (cap) |

Once a block expires the key disappears, and the next `Add` starts from the 1st call again.

## Block Record

`block:{ip}` holds an `IPItem` as JSON:

| Field | 1st call | Each later call |
|---|---|---|
| `reason` | This reason | Appends this reason after a newline |
| `added_at` | Now | Unchanged |
| `count` | 1 | Increments |
| `last` | Now | Updated to now |

## Zero-Value Risk

`BlockTimeMin` and `BlockTimeMax` have no defaults, and a Redis `SET` expiry of 0 means never expire:

| Setting | Result |
|---|---|
| `BlockTimeMin = 0` | Every block is permanent |
| `BlockTimeMin > 0`, `BlockTimeMax = 0` | The 1st block works; from the 2nd on the cap forces the duration to 0, making it permanent |

Set both to positive values.

To query or lift a block, see [Managing Blocks](/blocking-operations).

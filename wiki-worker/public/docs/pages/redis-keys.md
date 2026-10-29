# Redis Keys

This page lists the list, counter, and correlation keys go-ip-sentry writes to Redis with type, TTL, and writer, plus cleanup commands; no key has a prefix, so use a dedicated database index (`Redis.DB`), and `{sid}` is the session ID (without the signature), `{fp}` the device fingerprint, and `{minute}` the Unix minute.

## Lists

| Key | Type | TTL | Written by |
|---|---|---|---|
| `allow:{ip}` | string (`IPItem` JSON) | none | `Allow.Add`, startup load |
| `deny:{ip}` | string (`IPItem` JSON) | none | `Deny.Add`, startup load |
| `block:{ip}` | string (`IPItem` JSON) | block duration | `Block.Add` |
| `block:count:{ip}` | counter | 1 hour (from first write) | each request while blocked |

## Counters and Correlation

| Key | Type | TTL | Written by |
|---|---|---|---|
| `frequency:{ip}:{minute}` | counter | 2 minutes | every `Check` / `LoginFailure` / `NotFound404` |
| `session:ip:{sid}` | set | 1 hour (refreshed) | scoring |
| `ip:device:{ip}` | set | 1 hour (refreshed) | scoring |
| `device:fp:{fp}` | set | 1 hour (refreshed) | scoring |
| `fp:session:{minute}:{fp}` | set | 1 minute | scoring |
| `login:failure:{sid}` | counter | 1 hour (from first write) | `LoginFailure` |
| `notfound:404:{sid}` | counter | 1 hour (from first write) | `NotFound404` |

## Common Operations

```bash
redis-cli -n 2 DEL block:192.0.2.33
redis-cli -n 2 DEL allow:203.0.113.10
redis-cli -n 2 GET block:192.0.2.33
redis-cli -n 2 SCARD ip:device:203.0.113.5
```

After deleting an `allow:` / `deny:` key, also remove the entry from the list file and restart; see [Allow and Deny Lists](/access-lists).

Behavior and geo keys are in [Behavior and Geo Keys](/redis-keys-signals).

# Allow and Deny Lists

This page explains how the two permanent lists (Allow and Deny) compare, how they load at startup, how lookups work, and where they diverge across instances.

## Comparison

| Item | Allow | Deny |
|---|---|---|
| Access | `IPGuardian.Manager.Allow` | `IPGuardian.Manager.Deny` |
| On hit | `200`, skips every later check | `403`, `Device is banned, IP: {ip}` |
| Precedence | Highest; passes even when also blocked or denied | Checked after Block |
| Redis key | `allow:{ip}`, no expiry | `deny:{ip}`, no expiry |
| List file | `Filepath.WhiteList`, default `./whiteList.json` | `Filepath.BlackList`, default `./blackList.json` |
| Notification on add | None | Sends email asynchronously when `Email` is set; see [Email Alerts](/email-alerts) |

## Loading at Startup

`New()` reads each list file and writes every entry to both the memory cache and Redis (pipeline):

| Case | Behavior |
|---|---|
| File missing | Skipped |
| Read or JSON parse fails | Logs the error; `New()` still succeeds |
| Redis write fails | Logs the error; the memory cache is already loaded |

List file format (an `IPItem` array):

```json
[
  {"ip": "203.0.113.10", "reason": "office gateway", "added_at": 1735689600}
]
```

## Lookup

`Check(ip)` asks Redis with `EXISTS` first and returns `true` on a hit; on a miss or Redis error it falls back to the memory cache. Matching is exact string equality, so different IPv6 spellings (for example with zeros compressed) count as different IPs.

## Adding

```go
if err := sentry.Manager.Allow.Add("203.0.113.10", "office gateway"); err != nil {
	log.Printf("allow: %v", err)
}
if err := sentry.Manager.Deny.Add("198.51.100.7", "credential stuffing"); err != nil {
	log.Printf("deny: %v", err)
}
```

`Add` writes the memory cache, then Redis, then overwrites the list file with **this instance's memory cache**.

To add or remove entries, see [Managing Allow and Deny Lists](/access-lists-management).

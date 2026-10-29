# Removed API

This page lists public symbols removed or renamed across releases, the version that removed them, and their replacements, for upgrading from older versions.

## v0.3.0

v0.3.0 renamed the list managers to the Allow/Deny/Block tiers, unified list entries under `IPItem`, and switched to `pardnchiu/go-logger`.

### Types

| Old symbol | Removed | Last present | Replacement |
|---|---|---|---|
| `TrustManager` | Removed in v0.3.0 | v0.2.0 | `AllowIPManager` |
| `TrustItem` | Removed in v0.3.0 | v0.2.0 | `IPItem` |
| `BanManager` | Removed in v0.3.0 | v0.2.0 | `DenyIPManager` |
| `BanItem` | Removed in v0.3.0 | v0.2.0 | `IPItem` |
| `BlockManager` | Removed in v0.3.0 | v0.2.0 | `BlockIPManager` |
| `BlockItem` | Removed in v0.3.0 | v0.2.0 | `IPItem` (`Count`, `Last` fields) |
| `DeviceIS` | Removed in v0.3.0 | v0.2.0 | `IS` |
| `DeviceIP` | Removed in v0.3.0 | v0.2.0 | `IP` |
| `LogConfig` | Removed in v0.3.0 | v0.2.0 | `Log` (alias of `goLogger.Log`); `Config.Log` became `*Log` |

### Methods

| Old method | Removed | Last present | Replacement |
|---|---|---|---|
| `TrustManager.Add` | Removed in v0.3.0 | v0.2.0 | `AllowIPManager.Add` |
| `BanManager.Add` | Removed in v0.3.0 | v0.2.0 | `DenyIPManager.Add` |
| `BlockManager.Add` | Removed in v0.3.0 | v0.2.0 | `BlockIPManager.Add` |

### Signature and Field Changes

| Change | v0.2.0 | v0.3.0 onward |
|---|---|---|
| `New` parameter | `New(c *Config)` | `New(c Config)` |
| `Manager` fields | `Trust` / `Ban` / `Block` | `Allow` / `Deny` / `Block` |
| `Config.Log` | `LogConfig` (value; Removed in v0.3.0) | `*Log` (pointer; `nil` applies defaults) |
| `LogConfig.MaxSize` unit | MB (Removed in v0.3.0) | `Log.MaxSize` is in bytes |

### Upgrade Example

```go
// v0.2.0
sentry, err := golangIPSentry.New(&golangIPSentry.Config{ /* ... */ })
sentry.Manager.Trust.Add("203.0.113.10", "office")
sentry.Manager.Ban.Add("198.51.100.7", "abuse")

// v0.3.0 onward
sentry, err := golangIPSentry.New(golangIPSentry.Config{ /* ... */ })
sentry.Manager.Allow.Add("203.0.113.10", "office")
sentry.Manager.Deny.Add("198.51.100.7", "abuse")
```

No public symbols were removed between v0.1.0 and v0.2.0, or in v1.0.0.

# 已移除 API

本頁列出歷次版本移除或改名的公開符號、移除版本與替代寫法，供從舊版升級時對照。

## v0.3.0

v0.3.0 把名單管理器改名為 Allow／Deny／Block 三層、統一名單項目型別為 `IPItem`，並改用 `pardnchiu/go-logger`。

### 型別

| 舊符號 | 移除版本 | 最後存在 | 替代 |
|---|---|---|---|
| `TrustManager` | 移除於 v0.3.0 | v0.2.0 | `AllowIPManager` |
| `TrustItem` | 移除於 v0.3.0 | v0.2.0 | `IPItem` |
| `BanManager` | 移除於 v0.3.0 | v0.2.0 | `DenyIPManager` |
| `BanItem` | 移除於 v0.3.0 | v0.2.0 | `IPItem` |
| `BlockManager` | 移除於 v0.3.0 | v0.2.0 | `BlockIPManager` |
| `BlockItem` | 移除於 v0.3.0 | v0.2.0 | `IPItem`（`Count`、`Last` 欄位） |
| `DeviceIS` | 移除於 v0.3.0 | v0.2.0 | `IS` |
| `DeviceIP` | 移除於 v0.3.0 | v0.2.0 | `IP` |
| `LogConfig` | 移除於 v0.3.0 | v0.2.0 | `Log`（`goLogger.Log` 別名），`Config.Log` 改為 `*Log` |

### 方法

| 舊方法 | 移除版本 | 最後存在 | 替代 |
|---|---|---|---|
| `TrustManager.Add` | 移除於 v0.3.0 | v0.2.0 | `AllowIPManager.Add` |
| `BanManager.Add` | 移除於 v0.3.0 | v0.2.0 | `DenyIPManager.Add` |
| `BlockManager.Add` | 移除於 v0.3.0 | v0.2.0 | `BlockIPManager.Add` |

### 簽章與欄位變更

| 變更 | v0.2.0 | v0.3.0 起 |
|---|---|---|
| `New` 參數 | `New(c *Config)` | `New(c Config)` |
| `Manager` 欄位 | `Trust`／`Ban`／`Block` | `Allow`／`Deny`／`Block` |
| `Config.Log` | `LogConfig`（值；移除於 v0.3.0） | `*Log`（指標，`nil` 套用預設） |
| `LogConfig.MaxSize` 單位 | MB（移除於 v0.3.0） | `Log.MaxSize` 為位元組 |

### 升級範例

```go
// v0.2.0
sentry, err := golangIPSentry.New(&golangIPSentry.Config{ /* ... */ })
sentry.Manager.Trust.Add("203.0.113.10", "office")
sentry.Manager.Ban.Add("198.51.100.7", "abuse")

// v0.3.0 之後
sentry, err := golangIPSentry.New(golangIPSentry.Config{ /* ... */ })
sentry.Manager.Allow.Add("203.0.113.10", "office")
sentry.Manager.Deny.Add("198.51.100.7", "abuse")
```

`v0.2.0` 之前（v0.1.0 → v0.2.0）與 v1.0.0 沒有移除公開符號。

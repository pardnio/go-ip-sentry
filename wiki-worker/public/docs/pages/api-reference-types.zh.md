# API 型別

本頁列出套件 `golangIPSentry` 匯出的型別，區分呼叫端使用的主要型別與只供內部使用的型別。

## 主要型別

```go
type IPGuardian struct {
	Context  context.Context
	Config   *Config
	Redis    *redis.Client
	Logger   *Logger
	GeoLite2 *GeoLite2
	Manager  *Manager
}

type IPGuardianResult struct {
	Success    bool   `json:"success"`
	StatusCode int    `json:"status_code"`
	Error      string `json:"error"`
}

type IPItem struct {
	IP      string `json:"ip"`
	Reason  string `json:"reason"`
	AddedAt int64  `json:"added_at"`
	Count   int    `json:"count,omitempty"`
	Last    int64  `json:"last,omitempty"`
}
```

| 型別 | 說明 |
|---|---|
| `Config`、`Redis`、`Filepath`、`EmailConfig` | 設定，見 [設定](/zh/configuration) |
| `Parameter` | 門檻與分數，見 [參數](/zh/parameters) |
| `Log` | `goLogger.Log` 的別名 |
| `Logger` | `goLogger.Logger` 的別名，`IPGuardian.Logger` 可直接用來寫日誌 |
| `IPItem` | 名單檔與 Redis 中每筆名單的格式；`Count`／`Last` 只在 Block 使用 |

## 內部使用的匯出型別

下列型別匯出但沒有任何公開函式回傳或接收，屬實作細節，版本間可能變動：

| 型別 | 用途 |
|---|---|
| `Device`、`IS`、`IP` | `getDevice` 解析出的裝置、名單狀態與 IP 資訊 |
| `ScoreItem` | 單次評分結果（`IsBlock`、`IsSuspicious`、`IsDangerous`、`Flag`、`Score`、`Detail`、`Timestamp`） |
| `RiskScore` | 各維度累積的 `Base` 分數與 `Detail` |
| `BasicItem` | `calcBasic` 的關聯集合設定（欄位皆未匯出） |
| `ScoreTask`、`ScoreResult` | 目前未使用 |
| `GeoLite2` | GeoLite2 讀取器與快取（`IPGuardian.GeoLite2`，停用時為 `nil`） |
| `GeoLite2Config` | 目前未使用；路徑改由 `Filepath` 設定 |
| `Location` | GeoLite2 查詢結果（國家、城市、時區、經緯度、精度半徑） |

函式與方法見 [API 參考](/zh/api-reference)。

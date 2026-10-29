# API Types

This page lists the exported types of package `golangIPSentry`, separating the main types callers use from those exported only for internal use.

## Main Types

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

| Type | Description |
|---|---|
| `Config`, `Redis`, `Filepath`, `EmailConfig` | Settings; see [Configuration](/configuration) |
| `Parameter` | Thresholds and scores; see [Parameters](/parameters) |
| `Log` | Alias of `goLogger.Log` |
| `Logger` | Alias of `goLogger.Logger`; `IPGuardian.Logger` can be used to write logs |
| `IPItem` | Format of each list entry in files and Redis; `Count` / `Last` are used only by Block |

## Exported Types for Internal Use

These types are exported but no public function returns or accepts them; they are implementation details and may change between versions:

| Type | Purpose |
|---|---|
| `Device`, `IS`, `IP` | Device, list status, and IP data parsed by `getDevice` |
| `ScoreItem` | One scoring result (`IsBlock`, `IsSuspicious`, `IsDangerous`, `Flag`, `Score`, `Detail`, `Timestamp`) |
| `RiskScore` | A dimension's accumulated `Base` score and `Detail` |
| `BasicItem` | Correlation-set settings for `calcBasic` (all fields unexported) |
| `ScoreTask`, `ScoreResult` | Currently unused |
| `GeoLite2` | GeoLite2 readers and cache (`IPGuardian.GeoLite2`, `nil` when disabled) |
| `GeoLite2Config` | Currently unused; paths come from `Filepath` |
| `Location` | GeoLite2 lookup result (country, city, timezone, coordinates, accuracy radius) |

Functions and methods are in [API Reference](/api-reference).

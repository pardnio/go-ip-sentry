# API 參考

本頁列出套件 `golangIPSentry` 所有匯出的函式與方法：建立與釋放、請求檢查與整合、名單管理。

## 建立與釋放

| 函式 | 簽章 | 說明 |
|---|---|---|
| `New` | `func New(c Config) (*IPGuardian, error)` | 套用 Log／Redis 預設、建立 Logger、連線並 `PING` Redis、建立三個名單管理器並載入名單檔、開啟 GeoLite2 |
| `IPGuardian.Close` | `func (i *IPGuardian) Close() error` | 關閉 Redis、GeoLite2 與 Logger；Redis 關閉失敗時直接回傳錯誤，不再關閉其他資源 |

`New` 失敗情境：Logger 初始化失敗、Redis `PING` 失敗。名單檔與 GeoLite2 失敗只記錄，不會讓 `New` 失敗。

## 檢查與整合

| 方法 | 簽章 | 說明 |
|---|---|---|
| `IPGuardian.Check` | `func (i *IPGuardian) Check(r *http.Request, w http.ResponseWriter) IPGuardianResult` | 完整判定流程，見 [請求流程](/zh/request-lifecycle) |
| `IPGuardian.HTTPMiddleware` | `func (i *IPGuardian) HTTPMiddleware(next http.Handler) http.Handler` | `net/http` 中介層 |
| `IPGuardian.GinMiddleware` | `func (i *IPGuardian) GinMiddleware() gin.HandlerFunc` | Gin 中介層 |
| `IPGuardian.LoginFailure` | `func (i *IPGuardian) LoginFailure(w http.ResponseWriter, r *http.Request) error` | 當前 Session 登入失敗數加 1 |
| `IPGuardian.NotFound404` | `func (i *IPGuardian) NotFound404(w http.ResponseWriter, r *http.Request) error` | 當前 Session 404 數加 1 |

## 名單管理

透過 `IPGuardian.Manager`（型別 `Manager`，欄位 `Allow`、`Block`、`Deny`）存取：

| 方法 | 簽章 | 說明 |
|---|---|---|
| `AllowIPManager.Add` | `func (m *AllowIPManager) Add(ip string, tag string) error` | 寫入記憶體、Redis（無過期）、名單檔 |
| `AllowIPManager.Check` | `func (m *AllowIPManager) Check(ip string) bool` | 先查 Redis，失敗或不存在再查記憶體 |
| `DenyIPManager.Add` | `func (m *DenyIPManager) Add(ip, reason string) error` | 同 Allow，另外非同步寄送 Email |
| `DenyIPManager.Check` | `func (m *DenyIPManager) Check(ip string) bool` | 同 Allow |
| `BlockIPManager.Add` | `func (m *BlockIPManager) Add(ip string, reason string) error` | 暫時封鎖，重複呼叫時長倍增 |
| `BlockIPManager.IsBlock` | `func (m *BlockIPManager) IsBlock(ip string) bool` | 是否處於封鎖期；Redis 錯誤回 `false` |

`AllowIPManager` 與 `DenyIPManager` 的匯出欄位為 `Logger`、`Config`、`Redis`、`Context`、`Mutex`、`Cache`（`map[string]*IPItem`）；`BlockIPManager` 為 `Logger`、`Config`、`Redis`、`Context`。直接操作 `Cache` 須持有 `Mutex`。

結構定義與僅供內部使用的匯出型別見 [API 型別](/zh/api-reference-types)。

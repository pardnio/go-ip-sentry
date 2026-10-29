# 中介層

本頁說明 go-ip-sentry 的三種接法，以及如何接到 `net/http` 或 Gin。

## 三種接法

| 方式 | 簽章 | 失敗時 |
|---|---|---|
| `HTTPMiddleware` | `func (i *IPGuardian) HTTPMiddleware(next http.Handler) http.Handler` | 設定 `Content-Type: application/json`、寫入狀態碼與 `{"error": "..."}`，不呼叫 `next` |
| `GinMiddleware` | `func (i *IPGuardian) GinMiddleware() gin.HandlerFunc` | `c.JSON(status, gin.H{"error": ...})` 後 `c.Abort()` |
| `Check` | `func (i *IPGuardian) Check(r *http.Request, w http.ResponseWriter) IPGuardianResult` | 由呼叫端決定 |

三者在通過時都已寫入 Session 與裝置 Cookie，必須在 handler 寫出 body 之前執行。

## net/http

```go
mux := http.NewServeMux()
mux.HandleFunc("/api/orders", ordersHandler)

log.Fatal(http.ListenAndServeTLS(":8443", "cert.pem", "key.pem", sentry.HTTPMiddleware(mux)))
```

## Gin

```go
r := gin.New()
r.Use(gin.Recovery())

api := r.Group("/api")
api.Use(sentry.GinMiddleware())
api.GET("/orders", listOrders)

if err := r.RunTLS(":8443", "cert.pem", "key.pem"); err != nil {
	log.Fatal(err)
}
```

排除路徑或直接呼叫 `Check` 見 [中介層進階用法](/zh/middleware-advanced)。

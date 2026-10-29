# Middleware

This page shows the three ways to integrate go-ip-sentry and how to wire it into `net/http` or Gin.

## Three Ways to Integrate

| Option | Signature | On failure |
|---|---|---|
| `HTTPMiddleware` | `func (i *IPGuardian) HTTPMiddleware(next http.Handler) http.Handler` | Sets `Content-Type: application/json`, writes the status and `{"error": "..."}`, and skips `next` |
| `GinMiddleware` | `func (i *IPGuardian) GinMiddleware() gin.HandlerFunc` | `c.JSON(status, gin.H{"error": ...})` then `c.Abort()` |
| `Check` | `func (i *IPGuardian) Check(r *http.Request, w http.ResponseWriter) IPGuardianResult` | Up to the caller |

All three have already written the session and device cookies when they pass, so they must run before the handler writes a body.

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

To skip paths or call `Check` directly, see [Advanced Middleware Usage](/middleware-advanced).

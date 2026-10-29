# 中介層進階用法

本頁說明如何把特定路徑排除在中介層之外，以及直接呼叫 `Check` 自訂回應的寫法。

## 排除路徑

健康檢查、指標收集與固定週期輪詢會累積規律間隔與速率計數，應排除在外：

```go
func protect(sentry *golangIPSentry.IPGuardian, next http.Handler) http.Handler {
	guarded := sentry.HTTPMiddleware(next)
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		switch r.URL.Path {
		case "/healthz", "/metrics":
			next.ServeHTTP(w, r)
		default:
			guarded.ServeHTTP(w, r)
		}
	})
}
```

## 直接呼叫 Check

需要自訂回應格式（例如 HTML 錯誤頁）時：

```go
func handler(sentry *golangIPSentry.IPGuardian) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		result := sentry.Check(r, w)
		if !result.Success {
			http.Error(w, http.StatusText(result.StatusCode), result.StatusCode)
			return
		}
		w.Write([]byte("OK"))
	}
}
```

`IPGuardianResult.Error` 包含用戶端 IP，直接回給用戶端前先確認是否符合隱私需求。

`net/http` 與 Gin 的基本接法見 [中介層](/zh/middleware)。

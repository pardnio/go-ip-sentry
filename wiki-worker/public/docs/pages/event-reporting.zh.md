# 事件回報

本頁說明如何以 `LoginFailure` 與 `NotFound404` 把登入失敗與 404 掃描回報給評分引擎。

## 兩個回報方法

| 方法 | 遞增的 key | 參與評分 |
|---|---|---|
| `LoginFailure(w, r)` | `login:failure:{sid}` | 超過 `LoginFailure`（4）次開始加 `ScoreLoginFailure` |
| `NotFound404(w, r)` | `notfound:404:{sid}` | 超過 `NotFound404`（8）次開始加 `ScoreNotFound404` |

兩個計數以 Session 為單位，第一次遞增時設定 1 小時過期，之後不再延長（固定視窗）。計分規則見 [關聯訊號](/zh/correlation-signals)。

## 登入失敗

```go
func loginHandler(sentry *golangIPSentry.IPGuardian) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		if !validCredential(r) {
			if err := sentry.LoginFailure(w, r); err != nil {
				log.Printf("report login failure: %v", err)
			}
			http.Error(w, "Unauthorized", http.StatusUnauthorized)
			return
		}
		w.Write([]byte("welcome"))
	}
}
```

## 404 掃描

```go
r := gin.New()
r.Use(gin.Recovery(), sentry.GinMiddleware())
r.NoRoute(func(c *gin.Context) {
	if err := sentry.NotFound404(c.Writer, c.Request); err != nil {
		log.Printf("report 404: %v", err)
	}
	c.JSON(http.StatusNotFound, gin.H{"error": "not found"})
})
```

副作用與效果範圍見 [事件回報的副作用](/zh/event-reporting-effects)。

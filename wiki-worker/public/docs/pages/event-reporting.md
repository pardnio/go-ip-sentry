# Event Reporting

This page explains how to report login failures and 404 scans to the scoring engine with `LoginFailure` and `NotFound404`.

## Two Reporting Methods

| Method | Key incremented | Effect on scoring |
|---|---|---|
| `LoginFailure(w, r)` | `login:failure:{sid}` | Adds `ScoreLoginFailure` once the count exceeds `LoginFailure` (4) |
| `NotFound404(w, r)` | `notfound:404:{sid}` | Adds `ScoreNotFound404` once the count exceeds `NotFound404` (8) |

Both counters are per session; the first increment sets a one-hour expiry that is never extended afterward (a fixed window). Scoring rules are in [Correlation Signals](/correlation-signals).

## Login Failures

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

## 404 Scans

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

For side effects and scoring reach, see [Event Reporting Side Effects](/event-reporting-effects).

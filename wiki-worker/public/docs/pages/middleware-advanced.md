# Advanced Middleware Usage

This page shows how to keep specific paths out of the middleware and how to call `Check` directly for a custom response.

## Skipping Paths

Health checks, metrics scrapes, and fixed-period polling build up regular intervals and rate counts, so keep them out:

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

## Calling Check Directly

When you need a custom response format (an HTML error page, for example):

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

`IPGuardianResult.Error` contains the client IP; check your privacy requirements before echoing it back to clients.

For the basic `net/http` and Gin setup, see [Middleware](/middleware).

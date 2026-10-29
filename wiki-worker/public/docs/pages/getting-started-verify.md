# Verify the Setup

This page confirms the first server enforces the default rate limit, lists the files go-ip-sentry creates at runtime, and points to what to read next.

## Verify the Rate Limit

`curl` keeps no cookies by default, so every request is a new session, which makes the default per-minute limit easy to observe:

```bash
for i in $(seq 1 100); do
  curl -sk -o /dev/null -w "%{http_code}\n" https://localhost:8443/
done | sort | uniq -c
```

`RateLimitNormal` defaults to `100`; the count includes the current request and compares with `>=`, so the 100th request within the same minute returns `403`:

```json
{"error":"Device is reached rate limit (Normal), IP: 127.0.0.1"}
```

## Files Created at Runtime

| File | When |
|---|---|
| `.sessionSecret` | Created in the working directory on the first signed session (mode `0600`) |
| `./logs/mysqlPool*` | Default log path when `Log` is unset |
| `./whiteList.json` / `./blackList.json` | Written by `Allow.Add` / `Deny.Add` |

Add `.sessionSecret` to `.gitignore`; in multi-instance deployments every instance must share the same file, see [Session and Fingerprint](/session-fingerprint).

## Next Steps

- [Request Lifecycle](/request-lifecycle): when `Check` passes and when it returns `403`
- [Middleware](/middleware): Gin integration and skipping health-check paths
- [Parameters](/parameters): tune thresholds and scores

Installation and the first server code are in [Getting Started](/getting-started).

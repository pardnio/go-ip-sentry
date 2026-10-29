# Client IP Resolution

This page explains how go-ip-sentry picks the client IP from request headers, how it decides an address is internal, and the spoofing risk you must handle behind a reverse proxy.

## Resolution Order

Headers are read in this order; the value before the first comma is used if it parses as a valid IP:

| Order | Header | Typical source |
|---|---|---|
| 1 | `CF-Connecting-IP` | Cloudflare |
| 2 | `X-Forwarded-For` | Standard reverse proxy |
| 3 | `X-Real-IP` | Nginx |
| 4 | `X-Client-IP` | Apache |
| 5 | `X-Cluster-Client-IP` | Cluster load balancer |
| 6 | `X-Forwarded` | Legacy |
| 7 | `Forwarded-For` | Legacy |
| 8 | `Forwarded` | RFC 7239 (the `for=` syntax is not parsed; only a bare IP is accepted) |
| 9 | `RemoteAddr` | TCP peer |

## Spoofing Risk

Clients can set any of these headers, and the library has no trusted-proxy setting. In testing, a request sent with `X-Forwarded-For: 198.51.100.77` was handled as `198.51.100.77`, so:

| Attack | Result |
|---|---|
| A different forged IP on every request | Bypasses per-IP rate limits, Block, and Deny |
| Forge an IP on the Allow list | Skips every check |
| Forge someone else's IP | Raises their counters or even gets them blocked |

Pick at least one of these when deploying:

- Have the front proxy overwrite the headers: Nginx `proxy_set_header X-Forwarded-For $remote_addr;`, and strip the other listed headers
- Behind Cloudflare, accept origin connections only from Cloudflare IPs and strip every listed header except `CF-Connecting-IP`
- Without a proxy, delete all listed headers before the middleware so resolution falls through to `RemoteAddr`

## Internal Address Detection

These ranges count as internal, and internal IPs skip GeoLite2:

```
10.0.0.0/8   172.16.0.0/12   192.168.0.0/16   127.0.0.0/8
169.254.0.0/16   ::1/128   fc00::/7
```

Whether a request is internal depends only on `RemoteAddr`: if the direct peer is in these ranges the request is internal, so everything forwarded by an internal reverse proxy is marked internal; if the direct peer is not internal, the request never is. This result currently only feeds device information and does not affect scoring or admission; skipping GeoLite2 is decided separately from the resolved client IP.

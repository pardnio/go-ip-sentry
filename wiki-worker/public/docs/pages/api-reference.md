# API Reference

This page lists every exported function and method in package `golangIPSentry`: lifecycle, request checking and integration, and list management.

## Lifecycle

| Function | Signature | Description |
|---|---|---|
| `New` | `func New(c Config) (*IPGuardian, error)` | Applies Log/Redis defaults, creates the logger, connects and pings Redis, builds the three list managers and loads list files, opens GeoLite2 |
| `IPGuardian.Close` | `func (i *IPGuardian) Close() error` | Closes Redis, GeoLite2, and the logger; if closing Redis fails it returns that error without closing the rest |

`New` fails when the logger cannot initialize or the Redis ping fails. List-file and GeoLite2 failures are only logged and never fail `New`.

## Check and Integration

| Method | Signature | Description |
|---|---|---|
| `IPGuardian.Check` | `func (i *IPGuardian) Check(r *http.Request, w http.ResponseWriter) IPGuardianResult` | The full decision flow; see [Request Lifecycle](/request-lifecycle) |
| `IPGuardian.HTTPMiddleware` | `func (i *IPGuardian) HTTPMiddleware(next http.Handler) http.Handler` | `net/http` middleware |
| `IPGuardian.GinMiddleware` | `func (i *IPGuardian) GinMiddleware() gin.HandlerFunc` | Gin middleware |
| `IPGuardian.LoginFailure` | `func (i *IPGuardian) LoginFailure(w http.ResponseWriter, r *http.Request) error` | Increments the current session's login-failure count |
| `IPGuardian.NotFound404` | `func (i *IPGuardian) NotFound404(w http.ResponseWriter, r *http.Request) error` | Increments the current session's 404 count |

## List Management

Accessed through `IPGuardian.Manager` (type `Manager` with fields `Allow`, `Block`, `Deny`):

| Method | Signature | Description |
|---|---|---|
| `AllowIPManager.Add` | `func (m *AllowIPManager) Add(ip string, tag string) error` | Writes memory, Redis (no expiry), and the list file |
| `AllowIPManager.Check` | `func (m *AllowIPManager) Check(ip string) bool` | Redis first; memory on a miss or error |
| `DenyIPManager.Add` | `func (m *DenyIPManager) Add(ip, reason string) error` | Same as Allow, plus an asynchronous email |
| `DenyIPManager.Check` | `func (m *DenyIPManager) Check(ip string) bool` | Same as Allow |
| `BlockIPManager.Add` | `func (m *BlockIPManager) Add(ip string, reason string) error` | Temporary block; repeated calls grow the duration |
| `BlockIPManager.IsBlock` | `func (m *BlockIPManager) IsBlock(ip string) bool` | Whether the IP is blocked; `false` on a Redis error |

`AllowIPManager` and `DenyIPManager` export `Logger`, `Config`, `Redis`, `Context`, `Mutex`, and `Cache` (`map[string]*IPItem`); `BlockIPManager` exports `Logger`, `Config`, `Redis`, and `Context`. Hold `Mutex` before touching `Cache` directly.

Struct definitions and internal-only exported types are in [API Types](/api-reference-types).

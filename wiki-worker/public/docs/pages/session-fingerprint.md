# Session and Fingerprint

This page covers the two cookies go-ip-sentry issues, how the session ID is HMAC-signed and its key shared across instances, and the cookie boundaries to watch for.

## Two Cookies

| Cookie | Content | Lifetime | Attributes |
|---|---|---|---|
| `conn.sess.id` | `s:{id}.{signature}`, where `id` is 32 random characters | 30 days | `HttpOnly`, `Secure`, `SameSiteStrictMode`, `Path=/` |
| `conn.device.id` | 128 random characters, unsigned | 365 days | Same as above |

Both are rewritten on every `Check`, `LoginFailure`, and `NotFound404` call to refresh their lifetime.

## Session Signing

| Step | Behavior |
|---|---|
| Sign | `HMAC-SHA256(secret, id)`, base64url without trailing `=` |
| Verify | Cookie must start with `s:` and split on `.` into two parts; the signature is compared in constant time |
| Invalid or missing | Issues a new session silently |
| Key source | `.sessionSecret` in the working directory (mode `0600`); when missing or empty, a 128-character random value is generated and written |
| Load timing | Read once per process (`sync.Once`) |

### Multi-Instance Deployment

`.sessionSecret` uses a relative path. If each instance generates its own key, a session signed by instance A fails verification on instance B and gets replaced, which distorts `session:ip` and the other correlations. When running several instances:

- Give every instance the same `.sessionSecret` (mount one file or write it before startup)
- Start them in the same working directory, or make sure the file sits in each one's working directory
- Rotating the key invalidates every existing session

## Boundaries

| Case | Behavior |
|---|---|
| Plain HTTP (not localhost) | Browsers do not store `Secure` cookies, so every request is a new session and fingerprint and one IP quickly triggers `ip_multi_device` |
| Cross-site embedding (iframes, third-party requests) | `SameSiteStrictMode` withholds the cookies; same as above |
| Client forges `conn.device.id` | The device cookie is unsigned and can be anything; a forged value only changes the fingerprint, while the session still needs a valid signature |

For how the device fingerprint is built and multi-session fingerprint detection, see [Device Fingerprint](/session-fingerprint-device).

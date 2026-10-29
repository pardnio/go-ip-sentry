# Errors and Responses

This page explains how `IPGuardian.Check` behaves when Redis or a lookup fails, and the JSON body the middlewares return on rejection.

## Error-Handling Boundaries

| Case | Behavior |
|---|---|
| Allow/Deny Redis lookup fails | Falls back to the memory cache |
| Block Redis lookup fails | Treated as not blocked (fail-open) |
| Per-minute count fails | Count treated as `1` |
| A scoring dimension fails (Redis error during `calcBasic`, `calcBehavior`, `calcFingerprint`, or the geo history write) | `dynamicScore` returns `nil` with the error, `Check` logs it and then reads `score.IsBlock`, which panics with a nil-pointer dereference |
| GeoLite2 lookup fails | Logged and skipped; the geo score is `0` and scoring continues |

When Redis is unreachable, the list checks and counters above degrade, but `calcBasic` fails on its pipeline, so every non-allow-listed request reaches the panic. `net/http` recovers a handler panic per connection and drops the response; in Gin, register `gin.Recovery()` before `GinMiddleware` to turn it into a `500`.

## Response Format

`Check` returns `IPGuardianResult`; both middlewares turn a failure into JSON:

```json
{"error": "Device is banned, IP: 198.51.100.77"}
```

To call `Check` yourself, see [Middleware](/middleware).

For the check order and rate-limit tiers, see [Request Lifecycle](/request-lifecycle).

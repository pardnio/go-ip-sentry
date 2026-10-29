# Behavior and Geo Keys

This page lists the Redis keys written by behavior and geo scoring, with type, TTL, and writer, where `{sid}` is the session ID (without the signature) and `{ip}` the client IP.

## Behavior

| Key | Type | TTL | Written by |
|---|---|---|---|
| `interval:last:{sid}` | string (ms) | 1 hour (refreshed) | scoring |
| `interval:{sid}` | list (up to 10) | 1 hour (refreshed) | scoring |
| `session:start:{sid}` | string (ms) | 15 minutes (refreshed) | scoring |

## Geo

| Key | Type | TTL | Written by |
|---|---|---|---|
| `geo:ip:{ip}` | string (`Location` JSON) | 24 hours | GeoLite2 lookup |
| `geo:locations:{sid}` | list (up to 10) | 24 hours (refreshed) | scoring (only when the City database is loaded) |

List, counter, and correlation keys and cleanup commands are in [Redis Keys](/redis-keys).

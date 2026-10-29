# Risk Scoring

This page explains how the four detection dimensions compute in parallel, merge into a 0–100 score, and how that score picks the request's rate tier.

## Computation

`dynamicScore` starts one goroutine per dimension; each accumulates its own score, flags, and detail map, then merges under a mutex:

| Dimension | Function | Page |
|---|---|---|
| Correlation | `calcBasic` | [Correlation Signals](/correlation-signals) |
| Geo | `calcGeo` | [Geo Detection](/geo-detection) |
| Behavior | `calcBehavior` | [Behavior Signals](/behavior-signals) |
| Fingerprint | `calcFingerprint` | [Session and Fingerprint](/session-fingerprint) |

If any dimension returns an error, the whole score fails (see [Errors and Responses](/request-lifecycle-errors)).

## Merge Rule

```
total = sum of dimension scores
if the detail map has more than 4 keys, add 25
cap total at 100
```

The bonus counts detail keys, not flags: `geo_high_risk` and `geo_hopping` share the `geoCountries` / `countries` keys, so both firing counts once.

## Tiers

| Field | Condition | Default threshold | Effect |
|---|---|---|---|
| `IsBlock` | total `>= 100` | fixed | the request gets `403` |
| `IsDangerous` | total `>= ScoreDangerous` | `80` | `RateLimitDangerous` applies |
| `IsSuspicious` | total `>= ScoreSuspicious` | `50` | `RateLimitSuspicious` applies |
| Normal | otherwise | - | only `RateLimitNormal` applies |

The score is recomputed on every request and never carries over; lasting effects come only from each signal's counters and sets in Redis.

For every flag with its score and worked examples under default settings, see [Flags and Examples](/risk-scoring-examples).

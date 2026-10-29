# Flags and Examples

This page lists every risk flag with its dimension and score, and walks through how default settings add them up in concrete scenarios.

## Flags

| Flag | Dimension | Score |
|---|---|---|
| `session_multi_ip` | Correlation | `ScoreSessionMultiIP` (1× or 2×) |
| `ip_multi_device` | Correlation | `ScoreIPMultiDevice` (1× or 2×) |
| `device_multi_ip` | Correlation | `ScoreDeviceMultiIP` (1× or 2×) |
| `frequent_404_errors` / `excessive_404_errors` | Correlation | `ScoreNotFound404` (1× / 2×) |
| `frequent_login_failures` / `excessive_login_failures` | Correlation | `ScoreLoginFailure` (1× / 2×) |
| `interval_request` | Behavior | `ScoreIntervalRequest` |
| `extremely_regular` | Behavior | `ScoreIntervalRequest` × 1.5 |
| `too_frequent_requests` | Behavior | `ScoreFrequencyRequest` |
| `moderate_long_connection` / `long_connection` / `extremely_long_connection` | Behavior | `ScoreLongConnection` (1× / 1.5× / 2×) |
| `fp_multi_session` | Fingerprint | `ScoreFpMultiSession` |
| `geo_high_risk` | Geo | `ScoreGeoHighRisk` |
| `geo_hopping` | Geo | `ScoreGeoHopping` |
| `geo_frequent_switching` | Geo | `ScoreGeoFrequentSwitch` |
| `rapid_geo_change` | Geo | `ScoreGeoRapidChange` |

## Examples with Defaults

| Scenario | Triggers | Total | Result |
|---|---|---|---|
| Cookieless script, 13+ new fingerprints from one IP within an hour | `ip_multi_device` 2× | 40 | Normal tier |
| 7 login failures in one session | `excessive_login_failures` | 30 | Normal tier |
| 3 sessions on one fingerprint within a minute | `fp_multi_session` | 50 | Suspicious tier, limit 50 |
| All three plus a 2-hour session | four signals summing to 142 | capped at 100 | `403` |

`ScoreItem` and the flags are never returned from `Check`; to observe them, infer from Redis or change the code.

For how dimensions merge and how tiers are chosen, see [Risk Scoring](/risk-scoring).

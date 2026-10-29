# Score Parameters

This page lists the `Parameter` fields that set correlation thresholds and the scores added by correlation, behavior, fingerprint, and geo signals, with their JSON keys and defaults.

## Correlation Thresholds and Scores

| Threshold field | JSON key | Default | Score field | JSON key | Default |
|---|---|---|---|---|---|
| `SessionMultiIP` | `session_multi_ip` | `4` | `ScoreSessionMultiIP` | `score_session_multi_ip` | `25` |
| `IPMultiDevice` | `ip_multi_device` | `8` | `ScoreIPMultiDevice` | `score_ip_multi_device` | `20` |
| `DeviceMultiIP` | `device_multi_ip` | `4` | `ScoreDeviceMultiIP` | `score_device_multi_ip` | `15` |
| `LoginFailure` | `login_failure` | `4` | `ScoreLoginFailure` | `score_login_failure` | `15` |
| `NotFound404` | `not_found_404` | `8` | `ScoreNotFound404` | `score_not_found_404` | `15` |

## Behavior, Fingerprint, and Geo Scores

| Field | JSON key | Default | Page |
|---|---|---|---|
| `ScoreIntervalRequest` | `score_interval_request` | `25` | [Behavior Signals](/behavior-signals) |
| `ScoreFrequencyRequest` | `score_frequency_request` | `0` (no default) | [Behavior Signals](/behavior-signals) |
| `ScoreLongConnection` | `score_long_connection` | `15` | [Behavior Signals](/behavior-signals) |
| `ScoreFpMultiSession` | `score_fp_multi_session` | `50` | [Session and Fingerprint](/session-fingerprint) |
| `ScoreGeoHighRisk` | `score_geo_high_risk` | `30` | [Geo Detection](/geo-detection) |
| `ScoreGeoHopping` | `score_geo_hopping` | `15` | [Geo Detection](/geo-detection) |
| `ScoreGeoFrequentSwitch` | `score_geo_frequent_switch` | `20` | [Geo Detection](/geo-detection) |
| `ScoreGeoRapidChange` | `score_geo_rapid_change` | `25` | [Geo Detection](/geo-detection) |
| `HighRiskCountry` | `high_risk_country` | `[]` | [Geo Detection](/geo-detection) |

Default rules, rate limits, tiers, and blocking fields are in [Parameters](/parameters).

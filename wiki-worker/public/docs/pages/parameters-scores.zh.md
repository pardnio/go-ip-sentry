# 分數參數

本頁列出設定關聯門檻，以及關聯、行為、指紋、地理訊號加分的 `Parameter` 欄位、JSON key 與預設值。

## 關聯門檻與分數

| 門檻欄位 | JSON key | 預設 | 分數欄位 | JSON key | 預設 |
|---|---|---|---|---|---|
| `SessionMultiIP` | `session_multi_ip` | `4` | `ScoreSessionMultiIP` | `score_session_multi_ip` | `25` |
| `IPMultiDevice` | `ip_multi_device` | `8` | `ScoreIPMultiDevice` | `score_ip_multi_device` | `20` |
| `DeviceMultiIP` | `device_multi_ip` | `4` | `ScoreDeviceMultiIP` | `score_device_multi_ip` | `15` |
| `LoginFailure` | `login_failure` | `4` | `ScoreLoginFailure` | `score_login_failure` | `15` |
| `NotFound404` | `not_found_404` | `8` | `ScoreNotFound404` | `score_not_found_404` | `15` |

## 行為、指紋與地理分數

| 欄位 | JSON key | 預設 | 頁面 |
|---|---|---|---|
| `ScoreIntervalRequest` | `score_interval_request` | `25` | [行為訊號](/zh/behavior-signals) |
| `ScoreFrequencyRequest` | `score_frequency_request` | `0`（無預設） | [行為訊號](/zh/behavior-signals) |
| `ScoreLongConnection` | `score_long_connection` | `15` | [行為訊號](/zh/behavior-signals) |
| `ScoreFpMultiSession` | `score_fp_multi_session` | `50` | [Session 與指紋](/zh/session-fingerprint) |
| `ScoreGeoHighRisk` | `score_geo_high_risk` | `30` | [地理偵測](/zh/geo-detection) |
| `ScoreGeoHopping` | `score_geo_hopping` | `15` | [地理偵測](/zh/geo-detection) |
| `ScoreGeoFrequentSwitch` | `score_geo_frequent_switch` | `20` | [地理偵測](/zh/geo-detection) |
| `ScoreGeoRapidChange` | `score_geo_rapid_change` | `25` | [地理偵測](/zh/geo-detection) |
| `HighRiskCountry` | `high_risk_country` | `[]` | [地理偵測](/zh/geo-detection) |

預設值規則、速率、分級與封鎖欄位見 [參數](/zh/parameters)。

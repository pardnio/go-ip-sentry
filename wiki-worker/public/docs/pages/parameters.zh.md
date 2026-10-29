# 參數

本頁說明 `Parameter` 預設值的套用規則、速率、分級與封鎖欄位的 JSON key 與預設值，並提供常見情境的調整方向。

## 預設值規則

數值欄位為 `<= 0` 時套用預設；`BlockTimeMin`、`BlockTimeMax`、`HighRiskCountry` 沒有預設值。預設值在第一次用到該欄位的請求時才寫回 `Config`，不是在 `New()`。

## 速率與分級

| 欄位 | JSON key | 預設 | 作用 |
|---|---|---|---|
| `RateLimitNormal` | `rate_limit_normal` | `100` | 每 IP 每分鐘上限（所有請求） |
| `RateLimitSuspicious` | `rate_limit_suspicious` | `50` | 可疑分級的上限 |
| `RateLimitDangerous` | `rate_limit_dangerous` | `20` | 危險分級的上限 |
| `ScoreSuspicious` | `score_suspicious` | `50` | 可疑分級門檻 |
| `ScoreDangerous` | `score_dangerous` | `80` | 危險分級門檻 |
| `ScoreNormal` | `score_normal` | - | 未使用 |

## 封鎖

| 欄位 | JSON key | 預設 | 作用 |
|---|---|---|---|
| `BlockTimeMin` | `block_time_min` | 無（必填） | 首次封鎖時長 |
| `BlockTimeMax` | `block_time_max` | 無（必填） | 重複封鎖的時長上限 |
| `BlockToBan` | `block_to_ban` | `8` | 由 `Check` 寫入預設值；目前程式中沒有任何放行判定讀取它 |

為 0 的後果見 [暫時封鎖](/zh/blocking)。

## 調整方向

| 情境 | 建議 |
|---|---|
| 企業、校園等大型 NAT 出口 | 提高 `IPMultiDevice`，或將出口 IP 加入 Allow |
| 登入端點要對暴力破解更敏感 | 把 `LoginFailure` 降到 2–3，或把 `ScoreLoginFailure` 提高到 25 以上 |
| 公開 API 需要較高吞吐 | 提高 `RateLimitNormal`；可疑與危險上限保持較低 |
| 誤判偏多 | 先提高 `ScoreSuspicious`／`ScoreDangerous`，而不是逐項降低訊號分數 |

各訊號的門檻與分數見 [分數參數](/zh/parameters-scores)。

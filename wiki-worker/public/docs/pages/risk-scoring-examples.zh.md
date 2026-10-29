# Flag 與範例

本頁列出每個風險 flag 所屬維度與分數，並以預設值示範具體情境下的加總結果。

## Flag 一覽

| Flag | 維度 | 分數 |
|---|---|---|
| `session_multi_ip` | 關聯 | `ScoreSessionMultiIP`（1× 或 2×） |
| `ip_multi_device` | 關聯 | `ScoreIPMultiDevice`（1× 或 2×） |
| `device_multi_ip` | 關聯 | `ScoreDeviceMultiIP`（1× 或 2×） |
| `frequent_404_errors`／`excessive_404_errors` | 關聯 | `ScoreNotFound404`（1×／2×） |
| `frequent_login_failures`／`excessive_login_failures` | 關聯 | `ScoreLoginFailure`（1×／2×） |
| `interval_request` | 行為 | `ScoreIntervalRequest` |
| `extremely_regular` | 行為 | `ScoreIntervalRequest` × 1.5 |
| `too_frequent_requests` | 行為 | `ScoreFrequencyRequest` |
| `moderate_long_connection`／`long_connection`／`extremely_long_connection` | 行為 | `ScoreLongConnection`（1×／1.5×／2×） |
| `fp_multi_session` | 指紋 | `ScoreFpMultiSession` |
| `geo_high_risk` | 地理 | `ScoreGeoHighRisk` |
| `geo_hopping` | 地理 | `ScoreGeoHopping` |
| `geo_frequent_switching` | 地理 | `ScoreGeoFrequentSwitch` |
| `rapid_geo_change` | 地理 | `ScoreGeoRapidChange` |

## 預設值下的例子

| 情境 | 觸發 | 總分 | 結果 |
|---|---|---|---|
| 不帶 Cookie 的腳本，同一 IP 一小時內 13 個以上新指紋 | `ip_multi_device` 2× | 40 | 正常分級 |
| 同一 Session 7 次登入失敗 | `excessive_login_failures` | 30 | 正常分級 |
| 同一指紋一分鐘內 3 個 Session | `fp_multi_session` | 50 | 可疑分級，上限 50 |
| 上述三項加上 2 小時長連線 | 四項加總 142 | 上限 100 | `403` |

`ScoreItem` 與 flag 不會從 `Check` 回傳；需要觀察時只能從 Redis 推算或修改程式碼。

維度合併與分級規則見 [風險評分](/zh/risk-scoring)。

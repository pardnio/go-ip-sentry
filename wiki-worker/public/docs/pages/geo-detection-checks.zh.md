# 地理異常偵測

本頁列出四項地理異常偵測的觸發條件與分數，並說明不可能旅行的計算方式。

## 四項偵測

| Flag | 條件 | 分數 |
|---|---|---|
| `geo_high_risk` | 歷史紀錄的 `CountryCode` 列於 `HighRiskCountry`；從 `geo:locations` 還原的紀錄把國家碼放在 `Country`、`CountryCode` 為空，因此目前程式中此項不會命中 | `ScoreGeoHighRisk`（30） |
| `geo_hopping` | 最近 1 小時內超過 4 個不同國家 | `ScoreGeoHopping`（15） |
| `geo_frequent_switching` | 最近 1 小時內 `>= 4` 個城市、`>= 5` 筆紀錄，且相鄰紀錄城市切換超過 4 次 | `ScoreGeoFrequentSwitch`（20） |
| `rapid_geo_change` | 最新兩筆相隔不到 1 小時，且推算時速 `> 800` km/h，或 30 分鐘內移動 `> 500` km | `ScoreGeoRapidChange`（25） |

## 不可能旅行（Impossible Travel）

距離以 Haversine 公式、地球半徑 6371 km 計算，時速 = 距離 ÷ 時間差。800 km/h 約為民航機巡航速度，正常移動不會超過；VPN 切換節點、代理池輪換則會。

| 例子 | 距離 | 時間差 | 結果 |
|---|---|---|---|
| 台北 → 東京 | 約 2,100 km | 10 分鐘 | 觸發（時速與 30 分鐘規則皆成立） |
| 台北 → 台中 | 約 140 km | 20 分鐘 | 不觸發（420 km/h） |
| 台北 → 東京 | 約 2,100 km | 3 小時 | 不比對（超過 1 小時） |

GeoLite2 啟用條件、位置快取與歷史紀錄見 [地理偵測](/zh/geo-detection)。

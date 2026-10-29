# 風險評分

本頁說明四個偵測維度如何並行計算、合併成 0–100 分，以及分數如何決定請求的速率分級。

## 計算方式

`dynamicScore` 為四個維度各開一個 goroutine，每個維度累積自己的分數、flag 與明細（Detail），完成後以 mutex 合併：

| 維度 | 函式 | 頁面 |
|---|---|---|
| 關聯 | `calcBasic` | [關聯訊號](/zh/correlation-signals) |
| 地理 | `calcGeo` | [地理偵測](/zh/geo-detection) |
| 行為 | `calcBehavior` | [行為訊號](/zh/behavior-signals) |
| 指紋 | `calcFingerprint` | [Session 與指紋](/zh/session-fingerprint) |

任一維度回傳錯誤，整次評分即失敗（見 [錯誤與回應](/zh/request-lifecycle-errors)）。

## 合併規則

```
總分 = 各維度分數加總
若 Detail 的 key 數 > 4，總分再加 25
總分上限 100
```

Detail 以 key 計數而非 flag 數：`geo_high_risk` 與 `geo_hopping` 共用 `geoCountries`／`countries` 兩個 key，同時觸發時只算一次。

## 分級

| 欄位 | 條件 | 預設門檻 | 效果 |
|---|---|---|---|
| `IsBlock` | 總分 `>= 100` | 固定 | 該次請求回 `403` |
| `IsDangerous` | 總分 `>= ScoreDangerous` | `80` | 改用 `RateLimitDangerous` |
| `IsSuspicious` | 總分 `>= ScoreSuspicious` | `50` | 改用 `RateLimitSuspicious` |
| 正常 | 其餘 | - | 只受 `RateLimitNormal` 限制 |

分數每次請求重新計算，不累積到下一次；持久的效果只來自各訊號在 Redis 中的計數與集合。

所有 flag 的分數與預設值下的計算例子見 [Flag 與範例](/zh/risk-scoring-examples)。

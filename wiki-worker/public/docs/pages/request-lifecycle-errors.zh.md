# 錯誤與回應

本頁說明 Redis 或查詢失敗時 `IPGuardian.Check` 的行為，以及中介層拒絕請求時回傳的 JSON 內容。

## 錯誤處理邊界

| 情境 | 行為 |
|---|---|
| Allow／Deny 查詢 Redis 失敗 | 回退記憶體快取 |
| Block 查詢 Redis 失敗 | 視為未封鎖（fail-open） |
| 每分鐘計數失敗 | 計數視為 `1` |
| 任一評分維度失敗（`calcBasic`、`calcBehavior`、`calcFingerprint` 或寫入地理歷史時 Redis 出錯） | `dynamicScore` 回傳 `nil` 與錯誤，`Check` 記錄後讀取 `score.IsBlock`，觸發 nil pointer panic |
| GeoLite2 查詢失敗 | 記錄後略過，地理分數為 `0`，評分照常進行 |

Redis 無法連線時，上述名單判定與計數會降級，但 `calcBasic` 的 pipeline 會失敗，因此所有不在 Allow 名單的請求都會走到 panic。`net/http` 會在每條連線上 recover handler 的 panic 並中斷回應；Gin 需在 `GinMiddleware` 之前註冊 `gin.Recovery()`，把它轉成 `500`。

## 回應格式

`Check` 回傳 `IPGuardianResult`；兩個中介層把失敗結果轉成 JSON：

```json
{"error": "Device is banned, IP: 198.51.100.77"}
```

自行呼叫 `Check` 的寫法見 [中介層](/zh/middleware)。

判定順序與速率分級見 [請求流程](/zh/request-lifecycle)。

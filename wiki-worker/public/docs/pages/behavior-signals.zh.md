# 行為訊號

本頁說明 `calcBehavior` 如何以請求間隔的變異數辨識自動化腳本，以及如何對長時間 Session 計分。

## 請求間隔

| Key | 內容 | TTL |
|---|---|---|
| `interval:last:{sid}` | 上一次請求的毫秒時間戳 | 1 小時 |
| `interval:{sid}` | 最近 10 個間隔（毫秒），新值在前 | 1 小時 |

Session 至少累積 5 個間隔後才開始判定。計算母體變異數（單位 ms²）：

| Flag | 條件 | 分數 |
|---|---|---|
| `interval_request` | 變異數 `< 1000`，且平均間隔介於 500 ms 與 30 秒之間 | `ScoreIntervalRequest`（25） |
| `extremely_regular` | 變異數 `< 100`，且樣本數 `>= 8` | `ScoreIntervalRequest` × 1.5（37） |
| `too_frequent_requests` | 間隔 `< 500` ms 的次數 `>= 16` | `ScoreFrequencyRequest`（0） |

變異數 1000 ms² 約等於標準差 32 ms，人工操作幾乎不可能達到；固定 `sleep` 的輪詢腳本則會同時命中前兩項，合計 62 分，進入可疑分級。

## 長時間 Session

`session:start:{sid}` 在 Session 第一次出現時寫入當下時間，之後每個請求把 TTL 重設為 15 分鐘：

| Flag | Session 持續時間 | 分數 |
|---|---|---|
| `moderate_long_connection` | `> 1` 小時 | `ScoreLongConnection`（15） |
| `long_connection` | `> 2` 小時 | `ScoreLongConnection` × 1.5（22） |
| `extremely_long_connection` | `> 4` 小時 | `ScoreLongConnection` × 2（30） |

閒置超過 15 分鐘 key 即過期，下一個請求重新起算，所以只有「持續 15 分鐘內必有請求」的 Session 會累積時間。

## 邊界

| 情境 | 行為 |
|---|---|
| 單頁應用的固定週期輪詢（例如每 5 秒） | 間隔極規律，會被當成腳本；輪詢端點應排除在中介層外，見 [中介層](/zh/middleware) |
| 長時間開著的儀表板 | 只要每 15 分鐘內有請求，1 小時後開始加分 |
| 不保存 Cookie 的用戶端 | 每次都是新 Session，間隔與持續時間都不會累積 |

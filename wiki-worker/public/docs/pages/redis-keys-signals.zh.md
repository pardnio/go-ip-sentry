# 行為與地理 Key

本頁列出行為與地理評分寫入的 Redis key，含型別、TTL 與寫入時機；`{sid}` 為 Session ID（不含簽章），`{ip}` 為用戶端 IP。

## 行為

| Key | 型別 | TTL | 寫入 |
|---|---|---|---|
| `interval:last:{sid}` | string（毫秒） | 1 小時（每次刷新） | 評分 |
| `interval:{sid}` | list（最多 10 筆） | 1 小時（每次刷新） | 評分 |
| `session:start:{sid}` | string（毫秒） | 15 分鐘（每次刷新） | 評分 |

## 地理

| Key | 型別 | TTL | 寫入 |
|---|---|---|---|
| `geo:ip:{ip}` | string（`Location` JSON） | 24 小時 | GeoLite2 查詢 |
| `geo:locations:{sid}` | list（最多 10 筆） | 24 小時（每次刷新） | 評分（僅在載入 City 資料庫時） |

名單、計數與關聯 key 及清除指令見 [Redis Key](/zh/redis-keys)。

# 暫時封鎖

本頁說明 Block 的指數倍增時長、重複封鎖時的紀錄累加，以及 `BlockTimeMin`／`BlockTimeMax` 為 0 時的風險。

## 時長計算

```go
if err := sentry.Manager.Block.Add("192.0.2.33", "manual review"); err != nil {
	log.Printf("block: %v", err)
}
```

| 呼叫 | 紀錄 `count` | 時長 |
|---|---|---|
| 第 1 次（未在封鎖中） | 1 | `BlockTimeMin` |
| 第 n 次（仍在封鎖中，n ≥ 2） | n | `2^n × BlockTimeMin`，上限 `BlockTimeMax` |

以 `BlockTimeMin = 5m`、`BlockTimeMax = 24h` 為例（前 4 次為實測值）：

| 次數 | Redis TTL |
|---|---|
| 1 | 5m |
| 2 | 20m |
| 3 | 40m |
| 4 | 1h20m |
| 9 | 24h（上限） |

封鎖過期後 key 消失，下一次 `Add` 從第 1 次重新計算。

## 封鎖紀錄

`block:{ip}` 存放 `IPItem` JSON：

| 欄位 | 第 1 次 | 之後每次 |
|---|---|---|
| `reason` | 本次原因 | 以換行附加本次原因 |
| `added_at` | 當下時間 | 不變 |
| `count` | 1 | 加 1 |
| `last` | 當下時間 | 更新為當下 |

## 零值風險

`BlockTimeMin` 與 `BlockTimeMax` 沒有預設值，Redis `SET` 的過期時間為 0 代表永不過期：

| 設定 | 結果 |
|---|---|
| `BlockTimeMin = 0` | 每次封鎖都永久生效 |
| `BlockTimeMin > 0`、`BlockTimeMax = 0` | 第 1 次正常，第 2 次起時長被上限壓成 0，變成永久 |

兩者都必須設定為正值。

查詢與解除封鎖見 [管理封鎖](/zh/blocking-operations)。

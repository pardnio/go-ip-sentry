# Redis Key

本頁列出 go-ip-sentry 寫入 Redis 的名單、計數與關聯 key，含型別、TTL、寫入時機與清除指令；所有 key 都沒有前綴，建議使用獨立 DB 編號（`Redis.DB`），`{sid}` 為 Session ID（不含簽章），`{fp}` 為裝置指紋，`{minute}` 為 Unix 分鐘。

## 名單

| Key | 型別 | TTL | 寫入 |
|---|---|---|---|
| `allow:{ip}` | string（`IPItem` JSON） | 無 | `Allow.Add`、啟動載入 |
| `deny:{ip}` | string（`IPItem` JSON） | 無 | `Deny.Add`、啟動載入 |
| `block:{ip}` | string（`IPItem` JSON） | 封鎖時長 | `Block.Add` |
| `block:count:{ip}` | counter | 1 小時（首次） | 封鎖期間每個請求 |

## 計數與關聯

| Key | 型別 | TTL | 寫入 |
|---|---|---|---|
| `frequency:{ip}:{minute}` | counter | 2 分鐘 | 每次 `Check`／`LoginFailure`／`NotFound404` |
| `session:ip:{sid}` | set | 1 小時（每次刷新） | 評分 |
| `ip:device:{ip}` | set | 1 小時（每次刷新） | 評分 |
| `device:fp:{fp}` | set | 1 小時（每次刷新） | 評分 |
| `fp:session:{minute}:{fp}` | set | 1 分鐘 | 評分 |
| `login:failure:{sid}` | counter | 1 小時（首次） | `LoginFailure` |
| `notfound:404:{sid}` | counter | 1 小時（首次） | `NotFound404` |

## 常用操作

```bash
redis-cli -n 2 DEL block:192.0.2.33
redis-cli -n 2 DEL allow:203.0.113.10
redis-cli -n 2 GET block:192.0.2.33
redis-cli -n 2 SCARD ip:device:203.0.113.5
```

刪除 `allow:`／`deny:` key 後還需移除名單檔中的項目並重啟，見 [Allow 與 Deny 名單](/zh/access-lists)。

行為與地理 key 見 [行為與地理 Key](/zh/redis-keys-signals)。

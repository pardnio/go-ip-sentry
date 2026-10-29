# 管理封鎖

本頁說明如何查詢與解除 Block。

## 查詢與解除

| 動作 | 方式 |
|---|---|
| 查詢 | `sentry.Manager.Block.IsBlock(ip)`；Redis 錯誤時回 `false` |
| 解除 | 沒有 API；`redis-cli DEL block:{ip}` |
| 放行已封鎖 IP | 加入 Allow 名單即可，Allow 優先於 Block |

封鎖時長與封鎖紀錄的計算見 [暫時封鎖](/zh/blocking)。

# 事件回報的副作用

本頁說明呼叫 `LoginFailure` 與 `NotFound404` 的副作用，以及這些計數對風險評分的影響範圍。

## 副作用

兩個方法內部都會重新執行裝置識別：

| 副作用 | 影響 |
|---|---|
| `frequency:{ip}:{minute}` 再加 1 | 同一個請求被計入兩次每分鐘請求數 |
| 重新寫入兩個 `Set-Cookie` | 必須在 handler 寫出 body 之前呼叫 |
| 沒有 Cookie 的請求 | 計數記在本次回應新簽發的 Session；用戶端不保存該 Cookie 就不會影響下一個請求 |

## 效果範圍

預設值下，7 次登入失敗只得 30 分，不會單獨造成拒絕；要讓暴力破解直接觸發限制，降低 `LoginFailure` 或提高 `ScoreLoginFailure`，見 [參數](/zh/parameters)。攻擊者丟棄 Cookie 即換到新 Session，此時需靠 `ip:device` 等 IP 層訊號。

回報方法與使用範例見 [事件回報](/zh/event-reporting)。

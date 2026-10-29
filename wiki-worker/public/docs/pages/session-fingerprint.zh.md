# Session 與指紋

本頁說明 go-ip-sentry 簽發的兩個 Cookie、Session ID 的 HMAC 簽章與多實例金鑰共用，以及 Cookie 相關的邊界。

## 兩個 Cookie

| Cookie | 內容 | 有效期 | 屬性 |
|---|---|---|---|
| `conn.sess.id` | `s:{id}.{signature}`，`id` 為 32 字元隨機值 | 30 天 | `HttpOnly`、`Secure`、`SameSiteStrictMode`、`Path=/` |
| `conn.device.id` | 128 字元隨機值，未簽章 | 365 天 | 同上 |

兩者在每次 `Check`、`LoginFailure`、`NotFound404` 都會重新寫入以刷新有效期。

## Session 簽章

| 步驟 | 行為 |
|---|---|
| 簽章 | `HMAC-SHA256(secret, id)`，base64url 去除結尾 `=` |
| 驗證 | Cookie 必須以 `s:` 開頭、以 `.` 分成兩段，簽章以常數時間比對 |
| 驗證失敗或缺少 | 直接簽發新 Session，不回報錯誤 |
| 金鑰來源 | 工作目錄的 `.sessionSecret`（權限 `0600`）；不存在或為空時產生 128 字元隨機值並寫入 |
| 載入時機 | 每個行程只讀一次（`sync.Once`） |

### 多實例部署

`.sessionSecret` 以相對路徑讀寫，每個實例的工作目錄若各自產生金鑰，A 實例簽發的 Session 在 B 實例驗證失敗、被換成新 Session，導致 `session:ip` 等關聯失真。部署多個實例時：

- 所有實例使用同一份 `.sessionSecret`（掛載同一檔案或於啟動前寫入）
- 以相同工作目錄啟動，或確保該檔案位於各自的工作目錄
- 輪替金鑰會讓所有既有 Session 失效

## 邊界

| 情境 | 行為 |
|---|---|
| 純 HTTP（非 localhost） | 瀏覽器不儲存 `Secure` Cookie，每個請求都是新 Session 與新指紋，同一 IP 很快觸發 `ip_multi_device` |
| 跨網域嵌入（iframe、第三方請求） | `SameSiteStrictMode` 不送出 Cookie，同上 |
| 用戶端偽造 `conn.device.id` | 裝置 Cookie 未簽章，可任意指定；偽造值只影響指紋，Session 仍需正確簽章 |

裝置指紋的組成與指紋多 Session 偵測見 [裝置指紋](/zh/session-fingerprint-device)。

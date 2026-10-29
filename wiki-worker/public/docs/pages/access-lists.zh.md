# Allow 與 Deny 名單

本頁說明兩份永久名單（白名單 Allow、黑名單 Deny）的行為對照、啟動載入、查詢方式與多實例下的差異。

## 行為對照

| 項目 | Allow | Deny |
|---|---|---|
| 存取 | `IPGuardian.Manager.Allow` | `IPGuardian.Manager.Deny` |
| 命中結果 | `200`，跳過所有後續檢查 | `403`，`Device is banned, IP: {ip}` |
| 優先序 | 最高；同時在 Block 或 Deny 也放行 | 在 Block 之後判定 |
| Redis key | `allow:{ip}`，無過期 | `deny:{ip}`，無過期 |
| 名單檔 | `Filepath.WhiteList`，預設 `./whiteList.json` | `Filepath.BlackList`，預設 `./blackList.json` |
| 新增後通知 | 無 | 有設定 `Email` 時非同步寄信，見 [Email 通知](/zh/email-alerts) |

## 啟動載入

`New()` 讀取名單檔，每筆同時寫入記憶體快取與 Redis（pipeline）：

| 情況 | 行為 |
|---|---|
| 檔案不存在 | 略過 |
| 讀取或 JSON 解析失敗 | 記錄錯誤，`New()` 仍成功 |
| Redis 寫入失敗 | 記錄錯誤，記憶體快取已載入 |

名單檔格式（`IPItem` 陣列）：

```json
[
  {"ip": "203.0.113.10", "reason": "office gateway", "added_at": 1735689600}
]
```

## 查詢

`Check(ip)` 先查 Redis `EXISTS`；存在即回 `true`，不存在或 Redis 錯誤時再查記憶體快取。比對是字串完全相同，IPv6 的不同寫法（例如省略零）視為不同 IP。

## 新增

```go
if err := sentry.Manager.Allow.Add("203.0.113.10", "office gateway"); err != nil {
	log.Printf("allow: %v", err)
}
if err := sentry.Manager.Deny.Add("198.51.100.7", "credential stuffing"); err != nil {
	log.Printf("deny: %v", err)
}
```

`Add` 依序寫入記憶體快取、Redis，再把**本實例記憶體快取**整份覆寫回名單檔。

新增或移除項目見 [管理 Allow 與 Deny 名單](/zh/access-lists-management)。

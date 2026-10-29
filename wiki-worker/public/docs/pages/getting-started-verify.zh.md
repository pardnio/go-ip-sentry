# 驗證安裝

本頁驗證第一個伺服器套用預設速率限制、列出 go-ip-sentry 執行期產生的檔案，並指引接下來該讀的頁面。

## 驗證速率限制

`curl` 預設不保存 Cookie，每個請求都是新 Session，正好能觀察預設的每分鐘上限：

```bash
for i in $(seq 1 100); do
  curl -sk -o /dev/null -w "%{http_code}\n" https://localhost:8443/
done | sort | uniq -c
```

預設 `RateLimitNormal` 為 `100`，計數包含當前請求且以 `>=` 比較，同一分鐘內第 100 個請求回 `403`：

```json
{"error":"Device is reached rate limit (Normal), IP: 127.0.0.1"}
```

## 執行期產生的檔案

| 檔案 | 時機 |
|---|---|
| `.sessionSecret` | 第一次簽發 Session 時於工作目錄建立（權限 `0600`） |
| `./logs/mysqlPool*` | `Log` 未設定時的預設日誌路徑 |
| `./whiteList.json`／`./blackList.json` | 呼叫 `Allow.Add`／`Deny.Add` 時寫入 |

`.sessionSecret` 應加入 `.gitignore`；多實例部署時所有實例必須共用同一份，見 [Session 與指紋](/zh/session-fingerprint)。

## 下一步

- [請求流程](/zh/request-lifecycle)：`Check` 在何時放行、何時回 `403`
- [中介層](/zh/middleware)：Gin 整合與排除健康檢查路徑
- [參數](/zh/parameters)：調整門檻與分數

安裝與第一個伺服器的程式碼見 [快速開始](/zh/getting-started)。

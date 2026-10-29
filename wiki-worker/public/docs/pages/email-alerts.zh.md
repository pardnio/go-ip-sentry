# Email 通知

本頁說明 IP 加入 Deny 名單時的 SMTP 通知、SMTP 設定，以及預設與自訂的主旨、內文。

## 觸發時機

只有 `Deny.Add` 會寄信，且在寫入 Redis 與名單檔成功後以 goroutine 非同步執行；`Config.Email` 為 `nil` 時不寄送。寄送失敗只記錄錯誤，不影響 `Add` 的回傳值。

## 設定

```go
subject := func(ip, reason string) string {
	return "[Sentry] banned " + ip
}

sentry, err := golangIPSentry.New(golangIPSentry.Config{
	Redis: golangIPSentry.Redis{Host: "localhost", Port: 6379},
	Email: &golangIPSentry.EmailConfig{
		Host:     "smtp.example.com",
		Port:     587,
		Username: "alert@example.com",
		Password: "app-password",
		From:     "alert@example.com",
		To:       []string{"ops@example.com"},
		Subject:  &subject,
	},
	Parameter: golangIPSentry.Parameter{
		BlockTimeMin: 5 * time.Minute,
		BlockTimeMax: 24 * time.Hour,
	},
})
if err != nil {
	log.Fatal(err)
}
```

| 欄位 | 預設 |
|---|---|
| `Subject` | `[IP Sentry] IP {ip} has been banned` |
| `Body` | `[IP Sentry] IP {ip} has been banned for {reason}` |

自訂函式回傳空字串時回退預設值。`Subject`／`Body` 帶 `json:"-"`，從 JSON 載入設定時只能另外以程式指定。


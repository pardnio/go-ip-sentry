# Email Alerts

This page covers the SMTP alert sent when an IP joins the Deny list, its SMTP settings, and the default and custom subject and body.

## When It Fires

Only `Deny.Add` sends mail, asynchronously in a goroutine after the Redis and list-file writes succeed; nothing is sent when `Config.Email` is `nil`. A failed send is only logged and does not change what `Add` returns.

## Configuration

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

| Field | Default |
|---|---|
| `Subject` | `[IP Sentry] IP {ip} has been banned` |
| `Body` | `[IP Sentry] IP {ip} has been banned for {reason}` |

A custom function returning an empty string falls back to the default. `Subject` and `Body` carry `json:"-"`, so a JSON-loaded config must set them in code.


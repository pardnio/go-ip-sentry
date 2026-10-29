# 設定

本頁列出 `Config`、`Redis`、`Log` 的欄位、JSON key 與預設值，並示範從 JSON 載入完整設定；評分與速率參數另見 [參數](/zh/parameters)。

## Config

| 欄位 | JSON key | 型別 | 必要 | 說明 |
|---|---|---|---|---|
| `Redis` | `redis` | `Redis` | 是 | Redis 連線 |
| `Email` | `email` | `*EmailConfig` | 否 | Deny 通知；`nil` 不寄送 |
| `Log` | `log` | `*Log` | 否 | 日誌設定（`pardnchiu/go-logger`） |
| `Filepath` | `filepath` | `Filepath` | 否 | GeoLite2 與名單檔路徑 |
| `Parameter` | `parameter` | `Parameter` | 否 | 門檻、分數、速率、封鎖時長 |

`New()` 接收值而非指標，並保留自己的副本；建立後再修改呼叫端的 `Config` 不會生效。

## Redis

| 欄位 | JSON key | 預設 | 說明 |
|---|---|---|---|
| `Host` | `host` | `localhost` | 主機 |
| `Port` | `port` | `6379` | 連接埠；超出 1–65535 時套用預設 |
| `Password` | `password` | `""` | 密碼 |
| `DB` | `db` | `0` | DB 編號 |

所有 key 都沒有前綴，與其他服務共用 Redis 時建議使用獨立 DB 編號，key 一覽見 [Redis Key](/zh/redis-keys)。

## Log

`Log` 是 `pardnchiu/go-logger` 的 `Log` 型別別名：

| 欄位 | JSON key | 預設 | 說明 |
|---|---|---|---|
| `Path` | `path` | `./logs/mysqlPool` | 日誌路徑 |
| `Stdout` | `stdout` | `false` | 同時輸出到 stdout |
| `MaxSize` | `max_size` | `16777216`（16 MiB） | 單檔上限（位元組） |
| `MaxBackup` | `max_backups` | `5` | 保留的輪替檔數 |
| `Type` | `type` | `text` | `text` 或 `json` |

## 從 JSON 載入

```go
package main

import (
	"encoding/json"
	"log"
	"os"

	golangIPSentry "github.com/pardnchiu/golang-ip-sentry"
)

func main() {
	raw, err := os.ReadFile("sentry.json")
	if err != nil {
		log.Fatal(err)
	}
	var cfg golangIPSentry.Config
	if err := json.Unmarshal(raw, &cfg); err != nil {
		log.Fatal(err)
	}
	sentry, err := golangIPSentry.New(cfg)
	if err != nil {
		log.Fatal(err)
	}
	defer sentry.Close()
}
```

```json
{
  "redis": {"host": "localhost", "port": 6379, "db": 2},
  "log": {"path": "./logs/sentry", "type": "json"},
  "filepath": {"city_db": "./GeoLite2-City.mmdb", "trust_list": "./allow.json", "ban_list": "./deny.json"},
  "parameter": {"block_time_min": 300000000000, "block_time_max": 86400000000000}
}
```

`time.Duration` 欄位在 JSON 中以奈秒整數表示（上例為 5 分鐘與 24 小時）。

`Filepath` 與 `EmailConfig` 欄位見 [Filepath 與 EmailConfig](/zh/configuration-filepath-email)。

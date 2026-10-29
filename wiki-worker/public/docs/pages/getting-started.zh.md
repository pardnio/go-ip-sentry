# 快速開始

本頁說明安裝 go-ip-sentry、啟動 Redis，並以 `net/http` 跑出第一個受保護的伺服器。

## 前置需求

| 項目 | 需求 |
|---|---|
| Go | 1.24.3 以上（`go.mod`） |
| Redis | 任一可連線的實例；所有計數、名單與快取都存在 Redis |
| HTTPS | Session 與裝置 Cookie 帶 `Secure`，瀏覽器在純 HTTP 下不回傳，每個請求都會變成新 Session |
| GeoLite2（選用） | `GeoLite2-City.mmdb`，啟用地理偵測時才需要，見 [地理偵測](/zh/geo-detection) |

## 安裝

```bash
go get github.com/pardnchiu/golang-ip-sentry
```

模組路徑是 `github.com/pardnchiu/golang-ip-sentry`，套件名稱是 `golangIPSentry`，與目錄名不同，import 時明確指定別名。

## 啟動 Redis

```bash
docker run -d --name redis -p 6379:6379 redis:7-alpine
```

## 第一個伺服器

```go
package main

import (
	"log"
	"net/http"
	"time"

	golangIPSentry "github.com/pardnchiu/golang-ip-sentry"
)

func main() {
	sentry, err := golangIPSentry.New(golangIPSentry.Config{
		Redis: golangIPSentry.Redis{Host: "localhost", Port: 6379},
		Parameter: golangIPSentry.Parameter{
			// 必填：0 會讓封鎖紀錄永不過期
			BlockTimeMin: 5 * time.Minute,
			BlockTimeMax: 24 * time.Hour,
		},
	})
	if err != nil {
		log.Fatal(err)
	}
	defer sentry.Close()

	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("OK"))
	})

	log.Fatal(http.ListenAndServeTLS(":8443", "cert.pem", "key.pem", sentry.HTTPMiddleware(mux)))
}
```

`New()` 會先 `PING` Redis，連不上即回傳錯誤；成功時同時載入 Allow／Deny 名單檔（不存在則略過）。

下一步：[驗證安裝](/zh/getting-started-verify) 驗證預設速率限制並列出執行期產生的檔案。

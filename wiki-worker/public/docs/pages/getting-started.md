# Getting Started

This page installs go-ip-sentry, starts Redis, and runs a first protected server with `net/http`.

## Prerequisites

| Item | Requirement |
|---|---|
| Go | 1.24.3 or higher (`go.mod`) |
| Redis | Any reachable instance; every counter, list, and cache lives in Redis |
| HTTPS | Session and device cookies carry `Secure`, so browsers drop them over plain HTTP and every request becomes a new session |
| GeoLite2 (optional) | `GeoLite2-City.mmdb`, only for geo detection; see [Geo Detection](/geo-detection) |

## Installation

```bash
go get github.com/pardnchiu/golang-ip-sentry
```

The module path is `github.com/pardnchiu/golang-ip-sentry` and the package name is `golangIPSentry`, which differs from the directory name, so import it with an explicit alias.

## Start Redis

```bash
docker run -d --name redis -p 6379:6379 redis:7-alpine
```

## First Server

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
			// required: 0 makes block records never expire
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

`New()` pings Redis first and returns an error when it cannot connect; on success it also loads the Allow and Deny list files (skipped when absent).

Next: [Verify the Setup](/getting-started-verify) checks the default rate limit and lists the files created at runtime.

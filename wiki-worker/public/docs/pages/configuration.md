# Configuration

This page lists the fields, JSON keys, and defaults of `Config`, `Redis`, and `Log`, and shows loading the whole configuration from JSON; scoring and rate parameters are in [Parameters](/parameters).

## Config

| Field | JSON key | Type | Required | Description |
|---|---|---|---|---|
| `Redis` | `redis` | `Redis` | Yes | Redis connection |
| `Email` | `email` | `*EmailConfig` | No | Deny alerts; `nil` sends nothing |
| `Log` | `log` | `*Log` | No | Logger settings (`pardnchiu/go-logger`) |
| `Filepath` | `filepath` | `Filepath` | No | GeoLite2 and list file paths |
| `Parameter` | `parameter` | `Parameter` | No | Thresholds, scores, rates, block durations |

`New()` takes the config by value and keeps its own copy; changing the caller's `Config` afterward has no effect.

## Redis

| Field | JSON key | Default | Description |
|---|---|---|---|
| `Host` | `host` | `localhost` | Host |
| `Port` | `port` | `6379` | Port; outside 1–65535 the default applies |
| `Password` | `password` | `""` | Password |
| `DB` | `db` | `0` | Database index |

No key has a prefix, so use a dedicated database index when sharing Redis with other services; the key list is in [Redis Keys](/redis-keys).

## Log

`Log` is an alias of the `Log` type in `pardnchiu/go-logger`:

| Field | JSON key | Default | Description |
|---|---|---|---|
| `Path` | `path` | `./logs/mysqlPool` | Log path |
| `Stdout` | `stdout` | `false` | Also write to stdout |
| `MaxSize` | `max_size` | `16777216` (16 MiB) | Per-file limit in bytes |
| `MaxBackup` | `max_backups` | `5` | Rotated files kept |
| `Type` | `type` | `text` | `text` or `json` |

## Loading from JSON

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

`time.Duration` fields are nanosecond integers in JSON (5 minutes and 24 hours above).

`Filepath` and `EmailConfig` fields are in [Filepath and EmailConfig](/configuration-filepath-email).

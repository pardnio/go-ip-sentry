# Filepath 與 EmailConfig

本頁列出 `Filepath`（GeoLite2 與名單檔路徑）與 `EmailConfig`（Deny 通知的 SMTP 設定）的欄位、JSON key 與預設值。

## Filepath

| 欄位 | JSON key | 預設 | 說明 |
|---|---|---|---|
| `CityDB` | `city_db` | `""` | `GeoLite2-City.mmdb` 路徑 |
| `CountryDB` | `country_db` | `""` | `GeoLite2-Country.mmdb` 路徑 |
| `WhiteList` | `trust_list` | `./whiteList.json` | Allow 名單檔 |
| `BlackList` | `ban_list` | `./blackList.json` | Deny 名單檔 |

注意 JSON key 與欄位名不同（`trust_list`、`ban_list`）。

## EmailConfig

| 欄位 | JSON key | 型別 | 說明 |
|---|---|---|---|
| `Host` | `host` | `string` | SMTP 主機 |
| `Port` | `port` | `int` | SMTP 連接埠（建議 587） |
| `Username` | `username` | `string` | PLAIN 認證帳號 |
| `Password` | `password` | `string` | PLAIN 認證密碼 |
| `From` | `from` | `string` | 寄件者 |
| `To` | `to` | `[]string` | 收件者 |
| `CC` | `cc` | `[]string` | 只寫入標頭，不會實際投遞 |
| `Subject` | - | `*func(ip, reason string) string` | 自訂主旨 |
| `Body` | - | `*func(ip, reason string) string` | 自訂內文 |

細節見 [Email 通知](/zh/email-alerts)。

頂層 `Config`、`Redis`、`Log` 欄位見 [設定](/zh/configuration)。

# Filepath and EmailConfig

This page lists the fields, JSON keys, and defaults of `Filepath` (GeoLite2 and list file paths) and `EmailConfig` (SMTP settings for Deny alerts).

## Filepath

| Field | JSON key | Default | Description |
|---|---|---|---|
| `CityDB` | `city_db` | `""` | `GeoLite2-City.mmdb` path |
| `CountryDB` | `country_db` | `""` | `GeoLite2-Country.mmdb` path |
| `WhiteList` | `trust_list` | `./whiteList.json` | Allow list file |
| `BlackList` | `ban_list` | `./blackList.json` | Deny list file |

Note that the JSON keys differ from the field names (`trust_list`, `ban_list`).

## EmailConfig

| Field | JSON key | Type | Description |
|---|---|---|---|
| `Host` | `host` | `string` | SMTP host |
| `Port` | `port` | `int` | SMTP port (587 recommended) |
| `Username` | `username` | `string` | PLAIN auth user |
| `Password` | `password` | `string` | PLAIN auth password |
| `From` | `from` | `string` | Sender |
| `To` | `to` | `[]string` | Recipients |
| `CC` | `cc` | `[]string` | Header only; never delivered |
| `Subject` | - | `*func(ip, reason string) string` | Custom subject |
| `Body` | - | `*func(ip, reason string) string` | Custom body |

Details are in [Email Alerts](/email-alerts).

The top-level `Config`, `Redis`, and `Log` fields are in [Configuration](/configuration).

# Geo Detection

This page explains how the GeoLite2 databases are enabled, how IP locations are looked up, cached, and recorded per session, and the known boundaries of geo detection.

## Enabling

| Setting | Result |
|---|---|
| Both `Filepath.CityDB` and `Filepath.CountryDB` empty | No GeoLite2 instance; geo detection off |
| Both fail to open | Logs a warning; geo detection off |
| Only `CountryDB` opens | `calcGeo` requires `CityDB`, so geo detection is skipped entirely |
| `CityDB` opens | On; `CountryDB` is only a fallback when the City lookup fails |

Download the database (`GeoLite2-City.mmdb`) from MaxMind yourself; this library does not update it.

## Lookup and History

| Step | Behavior |
|---|---|
| Private IP | Skips the database and returns an empty location (no country or city, coordinates 0) |
| Cache | `geo:ip:{ip}` keeps the lookup result for 24 hours |
| Not found | Logged and skipped; the request's geo score is 0 |
| History | `geo:locations:{sid}` pushes `{ms}:{country code}:{city}:{lat}:{lng}`, keeps the last 10, TTL 24 hours |

## Boundaries

| Case | Behavior |
|---|---|
| One session alternating between private and public IPs | Private records sit at (0, 0), so switching to a public IP looks like a jump of thousands of kilometers from the Atlantic and may trigger `rapid_geo_change` |
| GeoLite2 accuracy | City-level locations can be off by tens of kilometers; different IPs in one city only affect the city-switch count |

For the four anomaly checks and impossible-travel examples, see [Geo Anomaly Checks](/geo-detection-checks).

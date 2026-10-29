# Geo Anomaly Checks

This page lists the trigger conditions and scores of the four geo anomaly checks and explains how impossible travel is calculated.

## Four Checks

| Flag | Condition | Score |
|---|---|---|
| `geo_high_risk` | A history record's `CountryCode` is listed in `HighRiskCountry`; records rebuilt from `geo:locations` carry the ISO code in `Country` and leave `CountryCode` empty, so this check does not match in the current code | `ScoreGeoHighRisk` (30) |
| `geo_hopping` | More than 4 distinct countries within the last hour | `ScoreGeoHopping` (15) |
| `geo_frequent_switching` | Within the last hour, `>= 4` cities, `>= 5` records, and more than 4 city switches between adjacent records | `ScoreGeoFrequentSwitch` (20) |
| `rapid_geo_change` | The two newest records are under an hour apart and the implied speed is `> 800` km/h, or the move is `> 500` km within 30 minutes | `ScoreGeoRapidChange` (25) |

## Impossible Travel

Distance uses the haversine formula with an Earth radius of 6371 km, and speed is distance divided by the time gap. 800 km/h is roughly airliner cruising speed, which normal movement never exceeds, while VPN node switches and rotating proxy pools do.

| Example | Distance | Gap | Result |
|---|---|---|---|
| Taipei → Tokyo | about 2,100 km | 10 minutes | Fires (both the speed and 30-minute rules hold) |
| Taipei → Taichung | about 140 km | 20 minutes | Does not fire (420 km/h) |
| Taipei → Tokyo | about 2,100 km | 3 hours | Not compared (over one hour) |

For enabling GeoLite2, location caching, and history storage, see [Geo Detection](/geo-detection).

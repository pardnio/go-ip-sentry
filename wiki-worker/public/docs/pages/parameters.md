# Parameters

This page covers how `Parameter` defaults apply, the rate, tier, and blocking fields with their JSON keys and defaults, and tuning directions for common situations.

## Default Rules

A numeric field `<= 0` receives its default; `BlockTimeMin`, `BlockTimeMax`, and `HighRiskCountry` have none. Defaults are written back into `Config` on the first request that uses the field, not in `New()`.

## Rates and Tiers

| Field | JSON key | Default | Applies to |
|---|---|---|---|
| `RateLimitNormal` | `rate_limit_normal` | `100` | Per-IP per-minute limit (every request) |
| `RateLimitSuspicious` | `rate_limit_suspicious` | `50` | Suspicious-tier limit |
| `RateLimitDangerous` | `rate_limit_dangerous` | `20` | Dangerous-tier limit |
| `ScoreSuspicious` | `score_suspicious` | `50` | Suspicious-tier threshold |
| `ScoreDangerous` | `score_dangerous` | `80` | Dangerous-tier threshold |
| `ScoreNormal` | `score_normal` | - | Unused |

## Blocking

| Field | JSON key | Default | Applies to |
|---|---|---|---|
| `BlockTimeMin` | `block_time_min` | none (required) | First block duration |
| `BlockTimeMax` | `block_time_max` | none (required) | Cap for repeated blocks |
| `BlockToBan` | `block_to_ban` | `8` | Default written by `Check`; no admission path reads it in the current code |

What 0 does is covered in [Temporary Blocking](/blocking).

## Tuning

| Situation | Suggestion |
|---|---|
| Large NAT egress (corporate, campus) | Raise `IPMultiDevice`, or allow-list the egress IP |
| Login endpoint should react faster to brute force | Lower `LoginFailure` to 2–3, or raise `ScoreLoginFailure` to 25 or more |
| Public API needs more throughput | Raise `RateLimitNormal`; keep the suspicious and dangerous limits low |
| Too many false positives | Raise `ScoreSuspicious` / `ScoreDangerous` first rather than lowering each signal's score |

Per-signal thresholds and scores are in [Score Parameters](/parameters-scores).

# Correlation Signals

This page explains how `calcBasic` tracks many-to-many links between sessions, IPs, and device fingerprints, and how login-failure and 404 counts are scored.

## Three Correlation Sets

Each request adds values to three Redis sets and resets each set's expiry to one hour:

| Set | Value added | Threshold field | Score field | Risk it signals |
|---|---|---|---|---|
| `session:ip:{sid}` | Client IP | `SessionMultiIP` (4) | `ScoreSessionMultiIP` (25) | One session seen from many IPs: hijacked session or rotating proxy pool |
| `ip:device:{ip}` | Device fingerprint | `IPMultiDevice` (8) | `ScoreIPMultiDevice` (20) | Many devices behind one IP: cookieless scripts or a large NAT |
| `device:fp:{fp}` | Client IP | `DeviceMultiIP` (4) | `ScoreDeviceMultiIP` (15) | One device seen from many IPs: copied device cookie or hopping proxies |

The expiry refreshes on every request, so "one hour" means one hour since the last request; a set that keeps receiving traffic never expires.

## Tiered Scoring

The three sets and the two counters below share one rule:

| Count | Points |
|---|---|
| `> threshold` | 1 × score |
| `> floor(threshold × 1.5)` | 2 × score |

With `SessionMultiIP = 4`, the 5th and 6th IPs add 25 and the 7th onward adds 50.

## Login Failures and 404s

| Counter key | Written by | Threshold field | Score field | Flag |
|---|---|---|---|---|
| `login:failure:{sid}` | `LoginFailure()` | `LoginFailure` (4) | `ScoreLoginFailure` (15) | `frequent_login_failures` / `excessive_login_failures` |
| `notfound:404:{sid}` | `NotFound404()` | `NotFound404` (8) | `ScoreNotFound404` (15) | `frequent_404_errors` / `excessive_404_errors` |

`calcBasic` only reads these counters and never increments them; see [Event Reporting](/event-reporting).

## Boundaries

| Case | Behavior |
|---|---|
| Client drops cookies on every request | Each request is a new session and fingerprint, so `session:ip` and `device:fp` never grow; `ip:device` grows with the new fingerprints instead |
| Many users behind one NAT | `ip:device` grows with the user count and scores from 9 devices; raise `IPMultiDevice` or allow-list corporate and campus egress IPs |
| Mobile network changes cells | IP changes accumulate in `session:ip` and `device:fp`; scoring starts at 5 IPs within an hour |
| Login failures count per session | An attacker resets the count by switching sessions; the server issues a fresh session whenever the cookie is dropped |

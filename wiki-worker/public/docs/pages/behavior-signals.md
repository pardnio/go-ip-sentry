# Behavior Signals

This page explains how `calcBehavior` spots automated scripts from the variance of request intervals, and how it scores long-lived sessions.

## Request Intervals

| Key | Content | TTL |
|---|---|---|
| `interval:last:{sid}` | Millisecond timestamp of the previous request | 1 hour |
| `interval:{sid}` | Last 10 intervals in milliseconds, newest first | 1 hour |

Scoring starts once a session has at least 5 intervals. It computes the population variance (ms²):

| Flag | Condition | Score |
|---|---|---|
| `interval_request` | variance `< 1000` and average interval between 500 ms and 30 s | `ScoreIntervalRequest` (25) |
| `extremely_regular` | variance `< 100` and `>= 8` samples | `ScoreIntervalRequest` × 1.5 (37) |
| `too_frequent_requests` | `>= 16` intervals under 500 ms | `ScoreFrequencyRequest` (0) |

A variance of 1000 ms² is a standard deviation of about 32 ms, which human browsing practically never reaches; a polling script with a fixed `sleep` hits both of the first two flags for 62 points, landing in the suspicious tier.

## Long Sessions

`session:start:{sid}` records the time a session first appears, and every later request resets its TTL to 15 minutes:

| Flag | Session duration | Score |
|---|---|---|
| `moderate_long_connection` | `> 1` hour | `ScoreLongConnection` (15) |
| `long_connection` | `> 2` hours | `ScoreLongConnection` × 1.5 (22) |
| `extremely_long_connection` | `> 4` hours | `ScoreLongConnection` × 2 (30) |

The key expires after 15 idle minutes and the next request starts over, so only sessions with a request at least every 15 minutes accumulate time.

## Boundaries

| Case | Behavior |
|---|---|
| Single-page app polling on a fixed period (say every 5 s) | Intervals are highly regular and look like a script; keep polling endpoints out of the middleware, see [Middleware](/middleware) |
| Dashboard left open for hours | Starts scoring after one hour as long as a request arrives every 15 minutes |
| Cookieless client | Every request is a new session, so neither intervals nor duration accumulate |

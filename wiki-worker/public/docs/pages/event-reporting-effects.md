# Event Reporting Side Effects

This page explains the side effects of calling `LoginFailure` and `NotFound404`, and how far their counts can move the risk score.

## Side Effects

Both methods run device identification again internally:

| Side effect | Impact |
|---|---|
| `frequency:{ip}:{minute}` increments again | The same request counts twice toward the per-minute limit |
| Both `Set-Cookie` headers are rewritten | Call before the handler writes a body |
| Request without cookies | The count lands on the session issued in this response; it only matters if the client keeps that cookie |

## Reach

With defaults, 7 login failures add only 30 points and never cause a rejection on their own; to make brute force hit the limit directly, lower `LoginFailure` or raise `ScoreLoginFailure`, see [Parameters](/parameters). An attacker who drops cookies moves to a new session, so IP-level signals such as `ip:device` have to carry the load.

For the reporting methods and usage examples, see [Event Reporting](/event-reporting).

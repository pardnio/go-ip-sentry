# Device Fingerprint

This page explains how the device fingerprint is derived from the User-Agent and device cookie, and when one fingerprint carrying several sessions gets flagged.

## Device Fingerprint

```
fingerprint = hex(SHA-256("{Platform}/{Browser}/{Type}/{OS}/{conn.device.id}"))
```

| Field | Parsing |
|---|---|
| Platform | User-Agent contains `android` / `iphone`, `ipad` / `windows` / `macintosh`, `mac os` / `linux` |
| Browser | Chrome (excluding Edge), Firefox, Safari (excluding Chrome), Edge, Opera |
| Type | Mobile keywords first, then tablet, otherwise Desktop |
| OS | iOS, Android, Windows 10/11, 8.1, 7, macOS versions, else falls back to Platform |

The fingerprint is tied to the device cookie, so clearing cookies yields a new fingerprint; keeping the device cookie but switching browsers or upgrading to a new major OS version also changes it.

## Fingerprint Multi-Session

`calcFingerprint` adds the session ID to `fp:session:{minute}:{fp}` (TTL 1 minute):

| Condition | Flag | Score |
|---|---|---|
| More than 2 sessions on one fingerprint within the same minute | `fp_multi_session` | `ScoreFpMultiSession` (50) |

Typical causes are a client that keeps the device cookie but keeps dropping the session cookie, or one device cookie copied into several parallel crawlers. This flag alone reaches the suspicious tier.

For the cookies and session signing, see [Session and Fingerprint](/session-fingerprint).

# 裝置指紋

本頁說明裝置指紋如何由 User-Agent 與裝置 Cookie 推導，以及同一指紋出現多個 Session 時何時被標記。

## 裝置指紋

```
fingerprint = hex(SHA-256("{Platform}/{Browser}/{Type}/{OS}/{conn.device.id}"))
```

| 欄位 | 解析方式 |
|---|---|
| Platform | User-Agent 含 `android`／`iphone`、`ipad`／`windows`／`macintosh`、`mac os`／`linux` |
| Browser | Chrome（排除 Edge）、Firefox、Safari（排除 Chrome）、Edge、Opera |
| Type | 先比對 mobile 關鍵字，再比對 tablet，其餘為 Desktop |
| OS | iOS、Android、Windows 10/11、8.1、7、macOS 版本，否則回退 Platform |

指紋綁定裝置 Cookie，所以同一台裝置清除 Cookie 後會變成新指紋；同一個裝置 Cookie 換瀏覽器或升級作業系統大版本也會改變指紋。

## 指紋多 Session

`calcFingerprint` 把 Session ID 加入 `fp:session:{minute}:{fp}`（TTL 1 分鐘）：

| 條件 | flag | 分數 |
|---|---|---|
| 同一分鐘內同一指紋超過 2 個 Session | `fp_multi_session` | `ScoreFpMultiSession`（50） |

典型情境是用戶端保留裝置 Cookie 卻不斷丟棄 Session Cookie，或同一份裝置 Cookie 被複製到多個並行的爬蟲。單獨觸發即達可疑分級。

Cookie 與 Session 簽章見 [Session 與指紋](/zh/session-fingerprint)。

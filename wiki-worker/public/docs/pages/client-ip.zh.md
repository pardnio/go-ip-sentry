# 用戶端 IP 解析

本頁說明 go-ip-sentry 如何從請求標頭決定用戶端 IP、內網判定方式，以及部署在反向代理後方時必須處理的偽造風險。

## 解析順序

依序讀取下列標頭，取第一個逗號前的值，能解析成合法 IP 即採用：

| 順序 | 標頭 | 常見來源 |
|---|---|---|
| 1 | `CF-Connecting-IP` | Cloudflare |
| 2 | `X-Forwarded-For` | 標準反向代理 |
| 3 | `X-Real-IP` | Nginx |
| 4 | `X-Client-IP` | Apache |
| 5 | `X-Cluster-Client-IP` | 叢集負載平衡 |
| 6 | `X-Forwarded` | 舊格式 |
| 7 | `Forwarded-For` | 舊格式 |
| 8 | `Forwarded` | RFC 7239（未解析 `for=` 語法，只接受純 IP） |
| 9 | `RemoteAddr` | TCP 連線來源 |

## 偽造風險

標頭由用戶端可任意設定，函式庫沒有「可信代理」設定。實測以 `X-Forwarded-For: 198.51.100.77` 發出的請求被當成 `198.51.100.77` 處理，因此：

| 攻擊 | 結果 |
|---|---|
| 每個請求帶不同的偽造 IP | 繞過每 IP 速率限制、Block 與 Deny |
| 偽造成已在 Allow 名單的 IP | 跳過所有檢查 |
| 偽造成他人 IP | 讓對方的計數上升，甚至被封鎖 |

部署時至少擇一：

- 前端代理覆寫標頭：Nginx `proxy_set_header X-Forwarded-For $remote_addr;`，並清除其他列出的標頭
- 經 Cloudflare 時只允許 Cloudflare IP 連到源站，並在源站清除 `CF-Connecting-IP` 以外的標頭
- 不經代理直接對外時，在中介層之前刪除上述所有標頭，讓解析落到 `RemoteAddr`

## 內網判定

下列網段視為內網，內網 IP 不查 GeoLite2：

```
10.0.0.0/8   172.16.0.0/12   192.168.0.0/16   127.0.0.0/8
169.254.0.0/16   ::1/128   fc00::/7
```

內網請求的判定只看 `RemoteAddr`：直接連線端屬於上述網段即為內網，因此經內網反向代理轉入的所有請求都會被標成內網；直接連線端不是內網時一律不是。此判定結果目前只寫入裝置資訊，不參與評分或放行；GeoLite2 是否略過則以解析出的用戶端 IP 另外判斷。

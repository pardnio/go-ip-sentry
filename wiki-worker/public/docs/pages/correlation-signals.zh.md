# 關聯訊號

本頁說明 `calcBasic` 如何追蹤 Session、IP 與裝置指紋之間的多對多關係，以及登入失敗與 404 計數如何計分。

## 三組關聯集合

每個請求把對應值加入三個 Redis 集合，並把集合的過期時間重設為 1 小時：

| 集合 | 加入的值 | 門檻欄位 | 分數欄位 | 代表的風險 |
|---|---|---|---|---|
| `session:ip:{sid}` | 用戶端 IP | `SessionMultiIP`（4） | `ScoreSessionMultiIP`（25） | 同一 Session 從多個 IP 出現：Session 被盜用或走代理池輪換 |
| `ip:device:{ip}` | 裝置指紋 | `IPMultiDevice`（8） | `ScoreIPMultiDevice`（20） | 同一 IP 出現大量裝置：不保存 Cookie 的腳本或大型 NAT |
| `device:fp:{fp}` | 用戶端 IP | `DeviceMultiIP`（4） | `ScoreDeviceMultiIP`（15） | 同一裝置從多個 IP 出現：裝置 Cookie 被複製或跨代理 |

過期時間在每次請求時刷新，所以「1 小時」是指距離最後一次請求 1 小時，持續有流量的集合不會過期。

## 分級計分

三組集合與下方兩個計數共用同一規則：

| 數量 | 加分 |
|---|---|
| `> 門檻` | 1 × 分數 |
| `> floor(門檻 × 1.5)` | 2 × 分數 |

以 `SessionMultiIP = 4` 為例：第 5、6 個 IP 加 25，第 7 個起加 50。

## 登入失敗與 404

| 計數 key | 寫入方式 | 門檻欄位 | 分數欄位 | flag |
|---|---|---|---|---|
| `login:failure:{sid}` | 呼叫 `LoginFailure()` | `LoginFailure`（4） | `ScoreLoginFailure`（15） | `frequent_login_failures`／`excessive_login_failures` |
| `notfound:404:{sid}` | 呼叫 `NotFound404()` | `NotFound404`（8） | `ScoreNotFound404`（15） | `frequent_404_errors`／`excessive_404_errors` |

`calcBasic` 只讀取這兩個計數，不會自動遞增；回報方式見 [事件回報](/zh/event-reporting)。

## 邊界

| 情境 | 行為 |
|---|---|
| 用戶端每個請求都丟棄 Cookie | 每次都是新 Session 與新指紋，`session:ip`、`device:fp` 不會累積；改由 `ip:device` 累積新指紋觸發 |
| 多個使用者共用 NAT 出口 | `ip:device` 會隨使用者數增加，超過 8 台裝置即加分；企業或校園網路應調高 `IPMultiDevice` 或加入 Allow 名單 |
| 行動網路切換基地台 | IP 變動會累積到 `session:ip` 與 `device:fp`，一小時內 5 個 IP 才開始加分 |
| 登入失敗以 Session 計算 | 攻擊者換 Session 即重新計數；Session 本身由伺服器簽發，丟棄 Cookie 就會拿到新的 |

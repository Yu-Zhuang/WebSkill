# 2021 dcard實習生 應徵作業
## Rate Limit Middleware

### I. 實測與使用

#### 1. 實測
* **step 1**
   * 下載rateLimitMiddleware.zp取得專案檔案
* **step 2** » 設定與啟動Redis (一行指令即可, 若無docker需先另外安裝docker)
```sh
sudo docker run -d -p 6379:6379 redis
```

* **step 3** » 至 ./bin 目錄下依照作業系統環境選擇啟動執行檔, 
**預設 rate limit num = 3 ; duration = 1 min 方便進行測試**, 需更改可在./conf/conf.go中更改後重新go build即可

* **step 4** » 以 http://127.0.0.1/ 造訪即可

#### 2. 使用
測試範例中將該middleware寫在 ```./controller/middleware.go```
若需要移植使用可參考範例檔案中middleware使用方式

---

### II. 實作工具
1. **Language**: Go
    * 選用Go原因:
        * 體質優良, 語法簡潔一致性高, 易讀
        * 部署方便
        * 其他: 目前正在進行中的畢業專題有在使用
2. **Database**: Redis
    * 選用原因:
        * 採用in-memory data structure store, **存取速度快**於傳統資料庫
        * 超**高併發**
        * 可減少主資料庫壓力
* reference: [redis offiical web](https://redis.io/)

---

### III. 實作特殊細節闡述
1. 資料夾存放內容註釋
    * bin: 執行程式的位元檔
    * conf: 參數設定
    * controller: 處理api的程序
    * dao: 連接資料庫
    * logic: 處理api所需的邏輯運算
    * models: 定義存入資料庫的結構
2. 減少不必要記憶體浪費: 
    * 如果存入資料庫的record在rate-reset時間到沒刪除, 在用戶量多時會造成浪費
    * 因此儲存redis方式使用```redis.Set(key, value, expire)```: 可使資料在時間到期自動刪除
3. middleware實作原始碼&註釋:
```go
func RateLimitMiddleware(c *gin.Context) {
    var user models.RateLimit
    user.IP = c.Request.RemoteAddr
    // check weather database have user record
    value := dao.DB.Get(user.IP)
    // if has user record (or not expire)
    if value.Err() == nil {
        // assign the ip's value which in database to user.RateLimitValue
        json.Unmarshal([]byte(value.Val()), &user.RateLimitValue)
        // if > rate limit
        if user.RemainNum <= 0 {
            // write res header , return 429 to user
            logic.WriteRateLimitHeader(c, strconv.Itoa(user.RemainNum), user.ExpireTime.String())
            c.JSON(http.StatusTooManyRequests, gin.H{
                "msg": "too many request",
            })
            c.Abort()
            return
        }
        // if not > rate limit
        user.RemainNum--
        b, _ := json.Marshal(&(user.RateLimitValue))
        dao.DB.Set(user.IP, string(b), user.ExpireTime.Sub(time.Now()))
        logic.WriteRateLimitHeader(c, strconv.Itoa(user.RemainNum), user.ExpireTime.String())
        // next()
        c.Next()
        return
    }
    // not has or expire: create new record to DB
    nUser := logic.CreateNewUserRateLimit()
    b, _ := json.Marshal(&(nUser.RateLimitValue))
    dao.DB.Set(user.IP, string(b), conf.RateLimitDuration)
    logic.WriteRateLimitHeader(c, strconv.Itoa(nUser.RemainNum), nUser.ExpireTime.String())
    // next()
    c.Next()
}
```

---

### IV. 聲明
1. 實作程式碼, 使用工具選擇, 演算法構思等 皆出自於作者本人.
2. 本小專題用於應徵工作, 可作參考但請勿抄襲程式碼去應徵.

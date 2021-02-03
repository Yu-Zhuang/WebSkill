# PWA 筆記

參考教學資源: [黑馬程序員教學影片](https://www.bilibili.com/video/BV1wt411E7QD?p=1)
## 功能
1. 如原生app般的桌面.app檔案, 打開也原生app般無瀏覽器網址顯示
2. 離線瀏覽內容
3. 跳提醒(兼容性較不佳, 可改用alert()或寄mail)

## 檔案
1. manifest.json : 設定該Web app加到桌面的一些屬性
2. service-worker.js : 可儲存cache至瀏覽器, 控制請求(request)往線上發送還是從cache中拿

## service-worker.js
1. install : 定義暫存什麼
2. activate : 設定清除快取
3. fetch : 控制請求往 cache 還是 網路
```javascript
const CACHE_NAME = "my_cache_v2"
// 緩存內容
self.addEventListener('install', async event => {
    console.log('install', event)
    // 開啟一個緩存
    const cache = await caches.open(CACHE_NAME)
    // 儲存資料在cache中
    await cache.addAll([
        '/',
        '/static/manifest.json',
        '/static/icons-144.png'
    ])
    await self.skipWaiting()
})

// 清除舊緩存
self.addEventListener('activate', async event => {
    console.log('activate', event)
    // 獲取所有cache的keys
    const keys = await caches.keys()
    // 走訪各個cache
    keys.forEach(k => {
        // 如果是舊的key則刪除
        if(k !== CACHE_NAME){
            caches.delete(k)
        }
    })
    await self.clients.claim()
})

// fetch
self.addEventListener('fetch', event => {
    const req = event.request
    event.respondWith(netWorkFirst(req))
})

async function netWorkFirst(req) {
    // 抓出請求
    try{
        const fresh = await fetch(req)
        return fresh
    } catch(e) {
        // 失敗了: 去讀取暫存
        console.log(`讀緩存`)
        const cache = await caches.open(CACHE_NAME) // 先打開瀏覽器中的暫存
        const cached = await caches.match(req) // 找出路徑對應的cache並回傳該暫存
        return cached
    }
    
}
```

## manifest.json

可以設定的欄位還很多, 但以下大致夠用
1. short_name : 加到桌面的應用名稱
2. icon : 加到桌面的圖片與打開時flash畫面的圖片
3. display : 設定是否需要瀏覽器網址那一槓, 有分三種
4. start_url : 設定開啟頁面

```json
{
    "short_name": "MyPWA",
    "name": "MyPWA",
    "description": "my pwa description",
    "icons": [
      {
        "src": "/static/192.png",
        "type": "image/png",
        "sizes": "192x192"
      },
      {
        "src": "/static/icons-144.png",
        "type": "image/png",
        "sizes": "144x144"
      }
    ],
    "start_url": "/",
    "background_color": "#fff", 
    "theme_color": "#000",
    "display": "standalone"
  }
```

---

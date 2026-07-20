---
date: 2026-07-20 08:17
title:
aliases:
tags:
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[【AJAX 傳輸】01. 觀念基礎篇：Content-Type 與 HTTP 傳輸]]
#### 📑 [[【AJAX 傳輸】02. 前後端實戰篇：$.ajax() 與 .NET 的收發攻防全解]]

---

# 【AJAX 傳輸】03. $.ajax() 完整範本（逐行註解版）

> **閱讀順序**：本篇是三部曲的最後一篇，是你未來開發時可以直接複製的「工具箱」。建議先讀完《01. 觀念基礎篇》和《02. 前後端實戰篇》，這裡的每一行註解你才會真正「有感」。

---

## 1. 完整範本（每一個指令都有註解）

```javascript
$.ajax({
    // ── 基本設定 ─────────────────────────────────────────────

    url: '/api/users',
    // 請求的目的地網址。
    // 在 MVC / Razor 中也常寫成：url: '@Url.Action("SaveUser", "User")'

    type: 'POST',
    // HTTP 方法。GET = 明信片（資料放網址、可分享、有長度限制）；
    //            POST = 密封紙箱（資料放 HTTP Body、容量大、網址乾淨）。
    // 🚨 不寫的話，預設值是 'GET'。
    // 補充：只有 POST、PUT、PATCH 等方法才會有 Body。

    // ── 發送方向：我要「寄」什麼格式 ─────────────────────────

    contentType: 'application/json',
    // 設定「要發送的數據類型」＝貼在包裹外箱的標籤，給「伺服器」看的。
    // 三種常見情境：
    //   (1) 不寫（預設）        → application/x-www-form-urlencoded（平信）
    //   (2) 'application/json'  → 傳 JSON，data 必須搭配 JSON.stringify()
    //   (3) false               → 檔案上傳（FormData）專用，讓瀏覽器自己
    //                             貼上帶有 boundary 隔板的 multipart/form-data
    // ⚠️ 標籤與內容必須一致：貼 JSON 標籤卻塞平信內容，後端直接 400。
    // ⚠️ 千萬不要手寫 'multipart/form-data'，會少了 boundary，後端拆不開便當盒。

    data: JSON.stringify({ name: 'John' }),
    // 要寄出的資料（包裹的內容物）。三種情境的正確搭配：
    //   (1) 平信   → data: { name: 'John' }
    //       jQuery 會自動用 $.param() 壓扁成 name=John
    //       ⚠️ 平信 + 陣列 + .NET 後端 → 記得加 traditional: true
    //   (2) JSON   → data: JSON.stringify({ name: 'John' })
    //       🚨 一定要自己 stringify！jQuery 不會自動幫你轉。
    //       忘記轉的下場：Payload 變成平信格式或 [object Object]，後端解析失敗。
    //   (3) 檔案   → data: formData（new FormData() 打包的便當盒）

    // traditional: true,
    // 只在「平信 + 陣列」時需要。
    // 叫 jQuery 用傳統方式序列化陣列：ids=1&ids=2&ids=3（.NET 看得懂），
    // 而不是預設的 ids[]=1&ids[]=2（PHP 風格，.NET 抓不到，Payload 會出現 %5B%5D）。
    // 用 JSON 傳送時完全不需要（stringify 後已是字串，不會啟動平信轉換）。

    // processData: false,
    // 魔法 1：叫 jQuery 閉嘴，不要把便當盒拆成平信 (a=1&b=2)。
    // 預設 true 的真實邏輯：「收到物件→轉平信；收到字串→直接放行」。
    // 所以 JSON（已 stringify 成字串）不用寫；FormData（物件）必須設 false，
    // 否則 jQuery 會試圖把便當盒壓扁，資料直接毀掉。

    // contentType: false,
    // 魔法 2：叫 jQuery 不要亂貼標籤，讓瀏覽器（郵差）自己貼上
    // 帶有 boundary 隔板的 multipart/form-data 標籤。
    // 瀏覽器在底層「認得」FormData 這個官方特製型別，會自動接手。
    // （注意：contentType 只能擇一設定——要嘛 'application/json'，要嘛 false，
    //   不能同時存在，這裡註解掉是示意檔案上傳時的寫法。）

    // ── 接收方向：我「預期收到」什麼格式 ─────────────────────

    dataType: 'json',
    // 設定「期望接收的數據類型」，決定 jQuery 怎麼解析伺服器的回應。
    // jQuery 的判斷優先順序：
    //   (1) 你明確指定的 dataType（最高優先級）
    //   (2) 沒指定 → 自動看伺服器回應的 Content-Type 判斷
    //   (3) 內容格式的智慧判斷
    // 回應標籤是 application/json 時，jQuery 會自動 JSON.parse 成物件，
    // 所以 success/done 裡可以直接寫 response.name。
    // 回應是 text/html 時，response 保持「字串」，適合直接 $('#div').html(response)。

    // ── 安全性設定 ───────────────────────────────────────────

    // headers: {
    //     RequestVerificationToken: $('[name="__RequestVerificationToken"]').val()
    // },
    // 【補充說明】自訂 HTTP 標頭。這裡是 .NET 的防偽驗證權杖（Anti-Forgery Token）：
    // 從頁面上的隱藏欄位取出 token，放進請求標頭，
    // 讓後端的 [ValidateAntiForgeryToken] 能驗證這是自家頁面發出的請求，
    // 防範 CSRF（跨站請求偽造）攻擊。
    // 後端有掛驗證屬性時，忘記帶 token 會直接收到 400。

    // ── 生命週期：請求前後的掛勾 ─────────────────────────────

    beforeSend: function(xhr) {
        // 請求「發送前」的處理，例如加入 loading 動畫。
        // xhr 參數是底層的 XMLHttpRequest 物件，也可在這裡補設定標頭。
        $('#loading').show();
    },

    complete: function() {
        // 請求「完成後」的處理（不論成功或失敗都會執行）。
        // 最適合放「關閉 loading 動畫」，保證不會有轉圈圈轉不停的窘境。
        $('#loading').hide();
    }

}).done(function(response) {
    // 請求「成功」的處理（等同於設定物件裡的 success，但鏈式寫法更清晰）。
    // response 已依 dataType / Content-Type 解析完成：
    // 若是 JSON，這裡拿到的就是可以直接 response.xxx 的物件。
    console.log('成功：', response);

}).fail(function(jqXHR, textStatus, errorThrown) {
    // 請求「失敗」的處理（等同於 error）。
    //   jqXHR       → 包含 status（如 400/500）、responseText（後端錯誤內容）
    //   textStatus  → 錯誤類型字串（"error"、"timeout"、"parsererror" 等）
    //   errorThrown → HTTP 錯誤說明（如 "Bad Request"）
    // 💡 除錯時搭配 F12 → Network → 點該請求 → Response 分頁看後端真正死因。
    console.log('失敗：', textStatus);

}).always(function() {
    // 「無論成功失敗」都會執行（時機在 done/fail 之後，功能類似 complete）。
    console.log('請求結束');
});
```

---

## 2. 三種情境的快速範本（直接複製用）

### 情境 A：平信（預設）——簡單查詢、表單送出

```javascript
$.ajax({
    url: '/api/searchUser',
    type: 'GET',                       // 查詢用 GET，網址可分享、可加書籤
    data: { dept: 'IT', name: 'john' } // jQuery 自動壓扁成 ?dept=IT&name=john
}).done(function(response) {
    console.log(response);
});
```

```javascript
// 平信 + 陣列（例如批次刪除）→ 必加 traditional
$.ajax({
    url: '/api/DeleteEmployees',
    type: 'POST',
    data: { department: 'IT', ids: [1, 2, 3] },
    traditional: true                  // 🚨 沒加的話 .NET 的 List<int> 會是空的
});
```

對應後端：

```csharp
[HttpPost]
public IActionResult DeleteEmployees(string department, List<int> ids)
// 平信不用加 [FromBody]！（或明確加 [FromForm]）
```

### 情境 B：JSON——複雜結構、多層物件、陣列

```javascript
$.ajax({
    url: '/api/saveUser',
    type: 'POST',
    contentType: 'application/json',       // (1) 貼標籤
    dataType: 'json',                      // 預期收到 JSON
    data: JSON.stringify({                 // (2) 壓扁成字串，缺一不可！
        username: 'john',
        hobbies: ['reading', 'coding'],
        preferences: { theme: 'dark' }
    })
}).done(function(response) {
    if (response.success) { alert(response.message); }
});
```

對應後端：

```csharp
[HttpPost]
public IActionResult SaveUser([FromBody] UserModel model)
// JSON 必加 [FromBody]，啟動 JSON 翻譯機
```

### 情境 C：檔案上傳——FormData 便當盒

```javascript
var formData = new FormData();
formData.append('username', 'john');
formData.append('hobbies', 'reading');   // 陣列：同名 key 重複 append
formData.append('hobbies', 'coding');
formData.append('preferences', JSON.stringify({ theme: 'dark' })); // JSON 塞夾層
formData.append('profilePhoto', $('#photoInput')[0].files[0]);     // 檔案

$.ajax({
    url: '/api/updateProfile',
    type: 'POST',
    data: formData,
    processData: false,   // 魔法 1：不要壓扁便當盒
    contentType: false    // 魔法 2：讓瀏覽器貼 multipart/form-data; boundary=...
}).done(function(response) {
    console.log('上傳成功');
});
```

對應後端：

```csharp
[HttpPost]
public IActionResult UpdateProfile(string username, List<string> hobbies,
                                   string preferences, IFormFile profilePhoto)
// 便當盒不用加 [FromBody]！檔案用 IFormFile 型別接
```

### 情境 D：檔案下載——不要用 AJAX！

```javascript
// 條件簡單 → 一行搞定，讓瀏覽器親自出馬，才會跳下載視窗
window.location.href = '@Url.Action("ExportToExcel", "Report")' + '?dept=IT';

// 條件複雜（大量 ID / JSON）→ 隱藏表單 POST，或進階用 fetch + Blob
// （詳見《02. 實戰篇》第 4 章）
```

---

## 3. 出手前的檢查清單 ✅

發出 AJAX 之前，快速自問：

1. **這次要傳什麼？** 純文字→平信（預設）；複雜結構→JSON；有檔案→FormData
2. **JSON 有沒有兩步到位？** `contentType: 'application/json'` + `JSON.stringify()`
3. **FormData 有沒有兩個 false？** `processData: false` + `contentType: false`
4. **平信傳陣列了嗎？** 有的話加 `traditional: true`
5. **後端指示詞對了嗎？** JSON→`[FromBody]`；表單/檔案→不寫或 `[FromForm]`；網址路徑→`[FromRoute]`；問號後面→`[FromQuery]`
6. **這是檔案下載嗎？** 是的話**不要用 AJAX**，改用 `location.href` 或隱藏表單
7. **出錯了？** F12 → Network → Fetch/XHR 過濾 → 看 Headers（標籤）、Payload（內容）、Response（死因）；必要時勾 Preserve log
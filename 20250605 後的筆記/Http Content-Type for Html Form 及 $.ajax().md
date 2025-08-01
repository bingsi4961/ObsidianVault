---
date: 2025-07-31 14:37
aliases: 
tags:
  - HTML
  - jQuery
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[]]
#### 📑 [[]]

---
## 第一章：基礎觀念建立

### 1.1 什麼是 Content-Type？

**Content-Type** 就像是一個**標籤**，告訴接收方：「我給你的這包東西是什麼格式」。

**生活化比喻**：

- 就像寄包裹時，你會在外面貼標籤寫「易碎品」、「食品」、「文件」
- Content-Type 就是告訴電腦：「這是 JSON 資料」、「這是 HTML 網頁」、「這是圖片檔案」

**技術定義**：

- Content-Type 是 HTTP Header 的一部分
- 用來描述 HTTP 訊息主體（Body）的資料格式
- 讓接收方知道如何正確處理資料

### 1.2 誰會設定 Content-Type？

#### 請求時（Request）

```html
<!-- 瀏覽器發送表單時自動設定 -->
<form method="post" action="/submit">
    <input type="text" name="username" value="john" />
    <input type="text" name="age" value="45" />
    
    <!-- 瀏覽器自動設定 Content-Type: application/x-www-form-urlencoded -->
    <!-- Http Body => username=john&age=45 -->
</form>
```

```javascript
// 開發者在 AJAX 中手動設定
$.ajax({
    contentType: 'application/json', // 我們告訴伺服器：我送的是 JSON
    data: JSON.stringify({name: 'john'})
});
```

#### 回應時（Response）

```csharp
// 伺服器設定回應的 Content-Type: application/json
public IActionResult GetData()
{
    return Json(new { name = "john" }); // 伺服器告訴瀏覽器：我回傳的是 JSON
}
```

### 1.3 為什麼 Content-Type 很重要？

**錯誤示範**：

```csharp
// 伺服器回傳 JSON 資料，但設定錯誤的 Content-Type
public IActionResult GetUser()
{
    var json = "{\"name\":\"john\",\"age\":30}";
    return Content(json, "text/plain"); // 錯誤：告訴瀏覽器這是純文字
}
```

```javascript
// 前端收到回應
$.ajax({
    url: '/GetUser',
    success: function(response) {
        console.log(typeof response); // "string" - 瀏覽器當作文字處理
        console.log(response.name);   // undefined - 無法直接存取屬性
        
        // 必須手動解析
        var userData = JSON.parse(response);
        console.log(userData.name);   // "john"
    }
});
```

**正確示範**：

```csharp
public IActionResult GetUser()
{
    return Json(new { name = "john", age = 30 }); // 正確：Content-Type 自動設為 application/json
}
```

```javascript
$.ajax({
    url: '/GetUser',
    success: function(response) {
        console.log(typeof response); // "object" - 瀏覽器自動解析成物件
        console.log(response.name);   // "john" - 可以直接存取屬性
    }
});
```

## 第二章：HTML 表單的 Content-Type

### 2.1 enctype 基礎概念

**enctype** = **enc**oding **type**（編碼類型）

```html
<!-- 完整語法 -->
<form method="post" action="/submit" enctype="編碼類型">
    <!-- 表單內容 -->
</form>
```

**重要觀念**：

- enctype **只對 POST 方法有效**
- GET 方法的資料永遠在 URL 中，不受 enctype 影響

### 2.2 第一種：application/x-www-form-urlencoded（預設）

#### 基本使用

```html
<!-- 這兩種寫法完全一樣 -->
<form method="post" action="/submit">
    <input type="text" name="username" value="john" />
    <input type="text" name="email" value="test@asus.com" />
    <input type="submit" value="送出" />
</form>

<!-- 明確寫出來 -->
<form method="post" action="/submit" enctype="application/x-www-form-urlencoded">
    <input type="text" name="username" value="john" />
    <input type="text" name="email" value="test@asus.com" />
    <input type="submit" value="送出" />
</form>
```

#### 實際傳送的內容

**HTTP Request**：

```
POST /submit HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 34

🚨 存在 Http Body 中
username=john&email=test%40asus.com
```

**注意編碼**：

- `@` 符號變成 `%40`
- 空格變成 `+` 或 `%20`
- `&` 符號變成 `%26`

#### 後端接收（.NET Core）

```csharp
[HttpPost]
public IActionResult Submit(string username, string email)
{
    // .NET Core 會自動解碼和 Model Binding
    Console.WriteLine($"使用者: {username}");     // "john"
    Console.WriteLine($"信箱: {email}");         // "test@asus.com"
    
    return View();
}
```

### 2.3 第二種：multipart/form-data（檔案上傳）

#### 基本概念

當表單包含檔案上傳時，**必須**使用這種格式：

```html
<!-- Content-Type: multipart/form-data; boundary=----WebKitFormBoundary...-->
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="text" name="username" value="john" />
    <input type="file" name="uploadFile" />
    <input type="submit" value="上傳" />
</form>
```

#### 為什麼需要 multipart/form-data？

**問題示範**（錯誤用法）：

```html
<!-- 錯誤：忘記設定 enctype -->
<form method="post" action="/upload">
    <input type="file" name="uploadFile" />
    <input type="submit" value="上傳" />
</form>
```

**結果**：檔案上傳會失敗，因為預設的 `application/x-www-form-urlencoded` 無法處理二進位檔案。

#### 實際傳送的內容

**HTTP Request**：( 瀏覽器負責產生和組織格式：包含 boundary 的產生 )

```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 2048

------WebKitFormBoundary7MA4YWxkTrZu0gW				← 開始標記
Content-Disposition: form-data; name="username"		← 欄位資訊

john												← 文字欄位的值
------WebKitFormBoundary7MA4YWxkTrZu0gW				← 分隔標記
Content-Disposition: form-data; name="uploadFile"; filename="report.pdf"
Content-Type: application/pdf						← 檔案類型

[這裡是 PDF 檔案的實際二進位內容]

------WebKitFormBoundary7MA4YWxkTrZu0gW--			← 結束標記 (注意最後有兩個--)
```

#### boundary 分隔符的作用

**理解 boundary**：

- `boundary` 是瀏覽器自動產生的唯一字串
- 用來分隔表單中的不同欄位
- 最後有兩個 `--`，表示所有資料結束

#### 後端接收（.NET Core）

```csharp
[HttpPost]
public IActionResult Upload(string username, IFormFile uploadFile)
{
	// .NET Core 自動處理 multipart/form-data 的解析
	// 你不需要手動處理 boundary

    Console.WriteLine($"使用者: {username}");
    
    if (uploadFile != null && uploadFile.Length > 0)
    {
        Console.WriteLine($"檔案名稱: {uploadFile.FileName}");
        Console.WriteLine($"檔案大小: {uploadFile.Length} bytes");
        Console.WriteLine($"檔案類型: {uploadFile.ContentType}");
        
        // 儲存檔案
        var filePath = Path.Combine("uploads", uploadFile.FileName);
        using (var stream = new FileStream(filePath, FileMode.Create))
        {
            uploadFile.CopyTo(stream);
        }
    }
    
    return View();
}
```

### 2.4 第三種：text/plain（很少使用）

#### 基本使用

```html
<form method="post" action="/submit" enctype="text/plain">
    <input type="text" name="username" value="john" />
    <input type="text" name="email" value="test@asus.com" />
    <input type="submit" value="送出" />
</form>
```

#### 實際傳送的內容

**HTTP Request**：

```
POST /submit HTTP/1.1
Content-Type: text/plain
Content-Length: 35

username=john
email=test@asus.com
```

#### 與 application/x-www-form-urlencoded 的比較

**text/plain 格式**：

```
username=john
email=test@asus.com
```

**application/x-www-form-urlencoded 格式**：

```
username=john&email=test%40asus.com
```

**主要差異**：

1. **分隔符號**：`\n`（換行）vs `&`
2. **編碼處理**：不編碼 vs URL 編碼
3. **解析難度**：需要手動解析 vs 框架自動解析

#### 為什麼很少使用？

1. **沒有標準**：沒有統一的解析規範
2. **處理困難**：需要手動解析，容易出錯
3. **不支援檔案**：無法處理二進位資料
4. **相容性差**：不是所有後端框架都支援

### 2.5 POST vs GET 的重要區別

#### GET 方法的特性

```html
<!-- GET 方法的表單 -->
<form method="get" action="/search">
    <input type="text" name="keyword" value="javascript" />
    <input type="submit" value="搜尋" />
</form>
```

**提交後的 URL**：

```
https://example.com/search?keyword=javascript
```

**重要觀念**：

- GET 方法的資料**永遠在 URL 中**
- enctype 對 GET 方法**無效**
- 資料會出現在瀏覽器歷史記錄中

#### POST 方法的特性

```html
<!-- POST 方法的表單 -->
<form method="post" action="/submit">
    <input type="text" name="username" value="john" />
    <input type="password" name="password" value="secret123" />
    <input type="submit" value="登入" />
</form>
```

**提交後的 URL**：

```
https://example.com/submit
```

（注意：URL 保持乾淨，沒有參數）

**重要觀念**：

- 🚨POST 方法的資料在 **HTTP Body 中** (username=john&password=secret123)
- URL 保持乾淨
- 敏感資料不會出現在 URL 中

## 第三章：jQuery AJAX 的 Content-Type

### 3.1 基本概念理解

在 jQuery AJAX 中，我們需要處理**兩個方向**的 Content-Type：

1. **我們發送給伺服器的資料格式**（Request Content-Type）
2. **伺服器回傳給我們的資料格式**（Response Content-Type）

### 3.2 發送資料給伺服器（Request）

#### 預設行為：application/x-www-form-urlencoded

```javascript
// 最簡單的 AJAX 請求
$.ajax({
    url: '/api/saveUser',
    type: 'POST',
    data: {
        username: 'john',
        email: 'john@asus.com',
        age: 30
    },
    success: function(response) {
        console.log('儲存成功');
    }
});
```

**實際發送的 HTTP Request**：

```
POST /api/saveUser HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=UTF-8

username=john&email=john%40asus.com&age=30
```

#### 發送 JSON 格式資料

```javascript
// 發送 JSON 格式
$.ajax({
    url: '/api/saveUser',
    type: 'POST',
    contentType: 'application/json',  // 告訴伺服器：我送的是 JSON
    data: JSON.stringify({           // 必須將物件轉成 JSON 字串
        username: 'john',
        email: 'john@asus.com',
        age: 30,
        hobbies: ['reading', 'coding']
    }),
    success: function(response) {
        console.log('儲存成功');
    }
});
```

#### 常見錯誤：忘記 JSON.stringify()

```javascript
// ❌ 錯誤：忘記轉換成 JSON 字串
$.ajax({
    contentType: 'application/json',
    data: {                          // 這裡是物件，不是 JSON 字串
        username: 'john',
        age: 30
    }
});

// ✅ 正確：手動轉換成 JSON 字串
$.ajax({
    contentType: 'application/json',
    data: JSON.stringify({           // 轉換成 JSON 字串
        username: 'john',
        age: 30
    })
});
```

#### 檔案上傳

```javascript
// 取得檔案
var selectedFile = $('#fileInput')[0].files[0];

// 建立 FormData
var formData = new FormData();
formData.append('username', 'john');
formData.append('uploadFile', selectedFile);

// 發送檔案
$.ajax({
    url: '/api/uploadFile',
    type: 'POST',
    data: formData,
    processData: false,    // 重要！告訴 jQuery 不要處理資料
    contentType: false,    // 重要！讓瀏覽器自動設定 Content-Type
    success: function(response) {
        console.log('上傳成功');
    }
});
```

==> **為什麼要設定 `contentType: false`？**

瀏覽器會自動產生完整的 Content-Type：

```
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
```

所以我們要設定 `contentType: false`，讓瀏覽器自動處理。

🚨 因為 7MA4YWxkTrZu0gW，是瀏覽器產生，我們自已沒辦法處理

==> **為什麼要設定 `processData: false`？** [[$.ajax() 的 processData 及 traditional]]

### 3.3 重要限制：檔案上傳不能用 JSON

#### 為什麼檔案不能用 JSON？

```javascript
// ❌ 這樣做是不可能的
var fileData = $('#fileInput')[0].files[0];
$.ajax({
    contentType: 'application/json',
    data: JSON.stringify({
        username: 'john',
        file: fileData  // 檔案是二進位資料，無法放入 JSON
    })
});
```

**原因**：

1. JSON 只能包含文字資料（字串、數字、布林值、陣列、物件）
2. 檔案是二進位資料，無法用 JSON 表示

#### 解決方案：使用 FormData (同時上傳 欄位值、JSON、檔案)

```javascript
// ✅ 正確做法：使用 FormData
var formData = new FormData();

// 一般資料
formData.append('username', 'john');

// 陣列資料 
formData.append('hobbies[]', 'reading'); 
formData.append('hobbies[]', 'coding');

// JSON 資料（轉成字串）
formData.append('preferences', JSON.stringify({
    theme: 'dark',
    language: 'zh-TW'
}));

// 檔案資料
formData.append('profilePhoto', $('#photoInput')[0].files[0]);

$.ajax({
    url: '/api/updateProfile',
    type: 'POST',
    data: formData,
    processData: false,
    contentType: false  🚨 // multipart/form-data
});
```

#### Http Body

```
POST /api/updateProfile HTTP/1.1
Host: localhost:44301
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 1234

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

john
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="hobbies[]"

reading
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="hobbies[]"

coding
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="preferences"

{"theme":"dark","language":"zh-TW"}
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="profilePhoto"; filename="avatar.jpg"
Content-Type: image/jpeg

[二進位檔案內容 - 實際的圖片資料]

------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

#### 後端接收 (.Net Core)

```csharp
[HttpPost]
public IActionResult UpdateProfile(string username, List<string> hobbies, string preferences, IFormFile profilePhoto)
{
    try
    {
        // 1. username 直接可用
        Console.WriteLine($"使用者名稱: {username}");
        
        // 2. 處理 hobbies 陣列
        if (hobbies != null && hobbies.Count > 0)
        {
            Console.WriteLine($"興趣愛好: {string.Join(", ", hobbies)}");
        }        
        
        // 3. preferences 是 JSON 字串，需要反序列化
        var preferencesObj = JsonSerializer.Deserialize<UserPreferences>(preferences);
        Console.WriteLine($"主題: {preferencesObj.Theme}, 語言: {preferencesObj.Language}");
        
        // 4. 處理上傳檔案
        if (profilePhoto != null && profilePhoto.Length > 0)
        {
            var fileName = Path.GetFileName(profilePhoto.FileName);
            var filePath = Path.Combine("uploads", fileName);
            
            using (var stream = new FileStream(filePath, FileMode.Create))
            {
                profilePhoto.CopyTo(stream);
            }
            
            Console.WriteLine($"檔案已上傳: {fileName}");
        }
        
        return Ok(new { 
            success = true, 
            message = "資料更新成功"
            }
        });
    }
    catch (Exception ex)
    {
        Console.WriteLine($"錯誤: {ex.Message}");
        return BadRequest(new { success = false, message = ex.Message });
    }
}
```

### 總結規則

|情況|contentType 設定|實際 Content-Type|
|---|---|---|
|一般表單資料|不設定 (預設)|`application/x-www-form-urlencoded`|
|JSON 資料|`'application/json'`|`application/json`|
|檔案上傳|`false`|`multipart/form-data; boundary=...`|

記住：**有檔案就用 FormData + contentType: false + processData: false** 

### 3.4 接收伺服器回應（Response）

#### jQuery 的自動解析機制

jQuery 會根據以下順序決定如何處理回應：

1. **你明確指定的 `dataType`**（最高優先級）
2. **伺服器回應的 `Content-Type`**
3. **內容格式的智慧判斷**

#### 範例一：JSON 回應

**伺服器端**：

```csharp
[HttpPost]
public IActionResult SaveUser(UserModel user)
{
    return Json(new { 
        success = true, 
        message = "儲存成功",
        userId = 123
    });
    
    // 自動設定 Content-Type: application/json
}
```

**前端接收**：

```javascript
$.ajax({
    url: '/api/saveUser',
    type: 'POST',
    data: { username: 'john', age: 30 },
    success: function(response) {
        // jQuery 自動解析 JSON，response 已經是 JavaScript 物件
        console.log(typeof response);        // "object"
        console.log(response.success);       // true
        console.log(response.message);       // "儲存成功"
        
        if (response.success) {
            alert('儲存成功！使用者 ID：' + response.userId);
        }
    }
});
```

#### 範例二：HTML 片段回應

**伺服器端**：

```csharp
public IActionResult GetUserCard(int userId)
{
    var html = $@"
        <div class='user-card'>
            <h3>使用者 {userId}</h3>
            <p>部門：IT</p>
            <button onclick='editUser({userId})'>編輯</button>
        </div>";
    
    return Content(html, "text/html");
}
```

**前端接收**：

```javascript
$.ajax({
    url: '/api/getUserCard',
    data: { userId: 123 },
    success: function(response) {
	    🚨 // $.ajax() 對 text/html 的處理是：不會自動解析成 DOM 物件，而是保持為字串。
        // response 是 HTML 字串
        console.log(typeof response);  // "string"
        
        // 直接插入到頁面中，jQuery 會自動解析 HTML
        $('#userContainer').html(response);
        // $('#userContainer').get(0).innerHTML = response;
    }
});
```

## 第四章：瀏覽器資源載入原理

### 4.1 CSS 載入詳細流程

#### 第一步：瀏覽器解析 HTML

```html
<!-- 瀏覽器讀到這行 -->
<link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css">
```

#### 第二步：發送 HTTP 請求

```
GET /css/bootstrap.min.css HTTP/1.1
Accept: text/css,*/*;q=0.1
```

#### 第三步：伺服器回應

```
HTTP/1.1 200 OK
Content-Type: text/css
Content-Length: 156789

/* Bootstrap CSS 內容 */
.container {
    max-width: 1140px;
    margin: 0 auto;
}

.btn {
    display: inline-block;
    padding: 6px 12px;
}
```

#### 第四步：瀏覽器處理 CSS

瀏覽器收到 CSS 後：

1. **解析 CSS 規則**
2. **建立 CSSOM（CSS 物件模型）**
3. **套用到 HTML 元素**

#### 重要概念：CSS 不會出現在 HTML 中

**查看頁面原始碼**（Ctrl+U）：

```html
<!-- 你只會看到這個 -->
<link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css">

<!-- 不會看到實際的 CSS 程式碼 -->
```

**CSS 存在哪裡？**

- 存在瀏覽器的**樣式表記憶體**中
- 可以透過 `document.styleSheets` 存取

```javascript
// 在瀏覽器 Console 中執行
console.log('載入的樣式表數量:', document.styleSheets.length);
```

### 4.2 JavaScript 載入詳細流程

#### 基本載入範例

```html
<!-- 瀏覽器讀到這行 -->
<script src="/js/app.js"></script>
```

#### 完整的載入和執行流程

**第一步：發送請求**

```
GET /js/app.js HTTP/1.1
Accept: application/javascript, text/javascript, */*
```

**第二步：伺服器回應**

```
HTTP/1.1 200 OK
Content-Type: application/javascript

console.log('app.js 載入完成');

function showWelcome() {
    alert('歡迎使用 ASUS Portal 系統！');
}

window.appVersion = '3.1.0';
```

**第三步：瀏覽器立即執行**

瀏覽器載入 JavaScript 後會**立即執行**：

1. **執行頂層程式碼**

```javascript
console.log('app.js 載入完成');  // 立即執行
```

2. **定義函數**（存在記憶體中）

```javascript
function showWelcome() { ... }   // 函數定義
```

3. **設定全域變數**

```javascript
window.appVersion = '3.1.0';     // 立即執行
```

#### 與 CSS 的重要差異

|特性|CSS|JavaScript|
|---|---|---|
|**處理時機**|載入後持續作用|載入時立即執行|
|**存在位置**|樣式表記憶體|全域作用域|
|**頁面影響**|改變視覺外觀|改變行為和內容|
|**HTML 變化**|不修改 HTML 結構|可能修改 HTML 結構|

### 4.3 在開發者工具中查看資源載入

#### F12 → Network 標籤

**查看所有載入的資源**：

```
Name                    Status  Type                   Size    Time
bootstrap.min.css       200     text/css               153KB   45ms
jquery-3.3.1.min.js     200     application/javascript  86KB   67ms
app.js                  200     application/javascript   12KB   23ms
```

#### F12 → Sources 標籤

**查看載入的原始碼**：

```
├── localhost:5000
    ├── css/
    │   ├── bootstrap.min.css    ← 點擊可查看 CSS 內容
    │   └── style.css
    ├── js/
    │   ├── jquery-3.3.1.min.js ← 點擊可查看 JavaScript 內容
    │   └── app.js
```

## 第五章：完整的 HTTP 請求結構

### 5.1 HTTP 請求的四個組成部分

每個 HTTP 請求都包含以下四個部分：

```
1. Request Line (請求行)
2. Request Headers (請求標頭)
3. Empty Line (空行)
4. Request Body (請求主體) - 可選
```

### 5.2 Request Line（請求行）

**格式**：`方法 路徑 HTTP版本`

**範例**：

```
POST /api/users HTTP/1.1
GET /images/logo.png HTTP/1.1
PUT /api/users/123 HTTP/1.1
DELETE /api/users/123 HTTP/1.1
```

### 5.3 Request Headers（請求標頭）

Headers 提供請求的額外資訊，格式為 `名稱: 值`。

#### 重要 Headers 說明

##### Host

```
Host: localhost:5000
```

**用途**：告訴伺服器要存取哪個網站

##### User-Agent

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/91.0.4472.124
```

**用途**：識別瀏覽器和作業系統

##### Content-Type

```
Content-Type: application/json; charset=utf-8
```

**用途**：<mark style="background: #FFF3A3A6;">說明請求 Body 的資料格式</mark>

##### Content-Length

```
Content-Length: 1234
```

**用途**：說明請求 Body 的大小（位元組）

##### Cookie

```
Cookie: sessionId=abc123; userPreference=darkMode
```

**用途**：傳送之前儲存的 Cookie 資料

### 5.4 Request Body（請求主體）

<mark style="background: #FFF3A3A6;">只有 POST、PUT、PATCH 等方法才會有 Body。</mark>

#### application/x-www-form-urlencoded 格式

**HTML 表單**：

```html
<form method="post" action="/login">
    <input name="username" value="john" />
    <input name="password" value="secret123" />
</form>
```

**完整的 HTTP 請求**：

```
POST /login HTTP/1.1
Host: localhost:5000
Content-Type: application/x-www-form-urlencoded
Content-Length: 32

username=john&password=secret123
```

#### application/json 格式

**AJAX 請求**：

```javascript
$.ajax({
    url: '/api/users',
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify({
        username: 'john',
        email: 'john@asus.com'
    })
});
```

**完整的 HTTP 請求**：

```
POST /api/users HTTP/1.1
Host: localhost:5000
Content-Type: application/json; charset=utf-8
Content-Length: 50

{"username":"john","email":"john@asus.com"}
```

#### multipart/form-data 格式

**檔案上傳表單**：

```html
<form method="post" action="/upload" enctype="multipart/form-data">
    <input name="title" value="我的報告" />
    <input name="file" type="file" />
</form>
```

**完整的 HTTP 請求**：

```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="title"

我的報告
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="report.pdf"
Content-Type: application/pdf

[PDF 檔案的二進位內容]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

## 第六章：常用 Content-Type 完整指南

### 6.1 Content-Type 使用時機分類

#### 主要用於請求（Request）

##### application/x-www-form-urlencoded

- **何時使用**：HTML 表單提交（預設）
- **資料格式**：`key=value&key=value`
- **編碼**：會進行 URL 編碼

##### multipart/form-data

- **何時使用**：檔案上傳或混合資料
- **資料格式**：使用 boundary 分隔
- **特殊要求**：必須設定 `enctype="multipart/form-data"`

##### application/json

- **何時使用**：API 呼叫、結構化資料傳送
- **資料格式**：JSON 字串
- **常見場景**：AJAX 請求、REST API

#### 主要用於回應（Response）

##### text/html

- **何時使用**：網頁內容
- **瀏覽器行為**：解析 HTML 標籤，建立 DOM，渲染頁面

##### text/css

- **何時使用**：CSS 樣式表
- **瀏覽器行為**：解析 CSS 規則，套用到元素

##### application/javascript

- **何時使用**：JavaScript 檔案
- **瀏覽器行為**：解析並執行 JavaScript 程式碼

##### application/json

- **何時使用**：API 回應、AJAX 資料
- **瀏覽器行為**：jQuery 自動解析成 JavaScript 物件

##### 檔案下載相關

**application/pdf**：

```csharp
return File(pdfBytes, "application/pdf", "report.pdf");
// 瀏覽器會嘗試顯示 PDF 或提示下載
```

**application/vnd.openxmlformats-officedocument.spreadsheetml.sheet**：

```csharp
// Excel 檔案 (.xlsx)
return File(excelBytes, 
    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", 
    "data.xlsx");
```

**application/octet-stream**：

```csharp
🚨 // 強制下載任何檔案
return File(fileBytes, "application/octet-stream", "file.zip");
```

### 6.2 Content-Type 對應表

|Content-Type|主要用於|說明|
|---|---|---|
|`application/x-www-form-urlencoded`|**請求**|HTML 表單預設格式|
|`multipart/form-data`|**請求**|檔案上傳必須用|
|`application/json`|**兩者**|API 資料交換|
|`text/plain`|**回應**|顯示純文字|
|`text/html`|**回應**|網頁內容|
|`text/css`|**回應**|樣式表檔案|
|`application/javascript`|**回應**|JavaScript 檔案|
|`application/pdf`|**回應**|PDF 檔案|
|`application/octet-stream`|**回應**|檔案下載|
|`image/png, image/jpeg`|**回應**|圖片檔案|

### 6.3 常見的 Content-Type 優先順序

##### 在你的開發中，使用頻率大概是：

-  **application/json** - API 開發最常用
-  **application/x-www-form-urlencoded** - 表單提交
-  **multipart/form-data** - 檔案上傳
-  **text/html** - 網頁回應
-  **application/vnd.openxmlformats-officedocument.spreadsheetml.sheet** - Excel 下載
-  **application/octet-stream** - 檔案下載

## 第七章：.NET Core 後端處理

### 7.1 接收不同格式的請求資料

#### 接收 form-urlencoded 資料

```csharp
[HttpPost]
public IActionResult SimpleForm(string username, string email, int age)
{
    // .NET Core 會自動進行 Model Binding
    Console.WriteLine($"使用者: {username}");
    Console.WriteLine($"信箱: {email}");
    Console.WriteLine($"年齡: {age}");
    
    return Json(new { success = true });
}
```

#### 接收 JSON 資料

```csharp
public class UserModel
{
    public string Username { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
}

[HttpPost]
public IActionResult ReceiveJson([FromBody] UserModel model)
{
    // [FromBody] 告訴 .NET Core 從 JSON 解析
    Console.WriteLine($"使用者: {model.Username}");
    Console.WriteLine($"信箱: {model.Email}");
    Console.WriteLine($"年齡: {model.Age}");
    
    return Json(new { success = true });
}
```

#### 接收檔案上傳

```csharp
[HttpPost]
public IActionResult UploadFile(string description, IFormFile uploadFile)
{
    Console.WriteLine($"描述: {description}");
    
    if (uploadFile != null && uploadFile.Length > 0)
    {
        Console.WriteLine($"檔案名稱: {uploadFile.FileName}");
        Console.WriteLine($"檔案大小: {uploadFile.Length} bytes");
        Console.WriteLine($"檔案類型: {uploadFile.ContentType}");
        
        // 儲存檔案
        var filePath = Path.Combine("uploads", uploadFile.FileName);
        using (var stream = new FileStream(filePath, FileMode.Create))
        {
            uploadFile.CopyTo(stream);
        }
        
        return Json(new { success = true, fileName = uploadFile.FileName });
    }
    
    return Json(new { success = false, message = "沒有選擇檔案" });
}
```

### 7.2 設定回應的 Content-Type

#### HTML 回應

```csharp
// 回傳 View (自動設定 text/html)
public IActionResult Dashboard()
{
    return View();
}

// 回傳 HTML 片段
public IActionResult GetUserCard(int userId)
{
    var html = $@"
        <div class='user-card'>
            <h3>使用者 {userId}</h3>
            <p>部門：IT</p>
        </div>";
    
    return Content(html, "text/html");
}
```

#### JSON 回應

```csharp
// 簡單 JSON 回應
public IActionResult GetUserList()
{
    var users = new[] {
        new { id = 1, name = "John", department = "IT" },
        new { id = 2, name = "Jane", department = "HR" }
    };
    
    return Json(users); // Content-Type: application/json
}
```

#### 檔案下載回應

```csharp
// Excel 檔案 (使用 ClosedXML)
public IActionResult ExportToExcel()
{
    using var workbook = new XLWorkbook();
    var worksheet = workbook.Worksheets.Add("資料");
    
    worksheet.Cell(1, 1).Value = "姓名";
    worksheet.Cell(1, 2).Value = "部門";
    worksheet.Cell(2, 1).Value = "John";
    worksheet.Cell(2, 2).Value = "IT";
    
    // 自動調整欄寬
    worksheet.ColumnsUsed().AdjustToContents();
    
    using var stream = new MemoryStream();
    workbook.SaveAs(stream);
    var excelBytes = stream.ToArray();
    
    return File(excelBytes, 
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", 
        "資料.xlsx");
}

// 強制下載任何檔案
public IActionResult ForceDownload(string fileName)
{
    var filePath = Path.Combine("files", fileName);
    if (!System.IO.File.Exists(filePath))
    {
        return NotFound();
    }
    
    var fileBytes = System.IO.File.ReadAllBytes(filePath);
    return File(fileBytes, "application/octet-stream", fileName);
}
```

## 第八章：開發者工具使用指南

### 8.1 F12 開發者工具各標籤

#### Network 標籤

- **用途**：查看所有 HTTP 請求和回應
- **重要欄位**：Name、Status、Type、Size、Time
- **查看詳細資訊**：Headers、Response、Request

#### Elements 標籤

- **用途**：查看和編輯 HTML DOM
- **Styles（樣式）**：查看套用的 CSS 規則
- **Computed（計算）**：查看最終的計算樣式

#### Sources 標籤

- **用途**：查看載入的資源原始碼
- **可以看到**：HTML、CSS、JavaScript 檔案內容

#### Console 標籤

- **用途**：查看 JavaScript 執行結果、執行程式碼、查看錯誤

### 8.2 實用檢查技巧

#### 檢查 AJAX 請求

1. Network 標籤勾選「Preserve log」
2. 篩選「XHR」只顯示 AJAX 請求
3. 點擊請求查看詳細資訊

## 第九章：常見錯誤和解決方案

### 9.1 Content-Type 相關錯誤

#### 檔案上傳失敗

```html
<!-- ❌ 錯誤：忘記設定 enctype -->
<form method="post" action="/upload">
    <input type="file" name="file" />
</form>

<!-- ✅ 正確 -->  
<form method="post" action="/upload" enctype="multipart/form-data">
    <input type="file" name="file" />
</form>
```

#### AJAX JSON 解析失敗

```csharp
// ❌ 錯誤：Content-Type 設定錯誤
public IActionResult GetData()
{
    var json = "{\"name\":\"john\"}";
    return Content(json, "text/plain");
}

// ✅ 正確：使用 Json() 方法
public IActionResult GetData()
{
    return Json(new { name = "john" });
}
```

#### AJAX 資料格式錯誤

```javascript
// ❌ 錯誤：說是 JSON 但傳送物件
$.ajax({
    contentType: 'application/json',
    data: { name: 'john' }
});

// ✅ 正確：轉換成 JSON 字串
$.ajax({
    contentType: 'application/json',
    data: JSON.stringify({ name: 'john' })
});
```

### 9.2 除錯流程

#### 前端除錯步驟

1. **F12 → Network**：檢查請求是否發送
2. **查看 Request Headers**：確認 Content-Type
3. **查看 Request Body**：確認資料格式
4. **查看 Response**：確認伺服器回應

#### 後端除錯技巧

```csharp
[HttpPost]
public IActionResult DebugRequest()
{
    Console.WriteLine($"Method: {Request.Method}");
    Console.WriteLine($"Content-Type: {Request.ContentType}");
    Console.WriteLine($"Content-Length: {Request.ContentLength}");
    
    // 輸出所有 Headers
    foreach (var header in Request.Headers)
    {
        Console.WriteLine($"{header.Key}: {header.Value}");
    }
    
    return Json(new { debug = "completed" });
}
```

## 第十章：實際應用場景

### 10.1 Portal 系統 (.NET Core 3.1)

#### AJAX 資料載入

```javascript
// jQuery 3.3.1 語法
function loadEmployeeData(empId) {
    $.ajax({
        url: '/api/employee/' + empId,
        type: 'GET',
        dataType: 'json',
        success: function(data) {
            $('#empName').text(data.name);
            $('#empDept').text(data.department);
        }
    });
}
```

#### ClosedXML 0.95.2 Excel 匯出

```csharp
public IActionResult ExportEmployee()
{
    using var workbook = new XLWorkbook();
    var worksheet = workbook.Worksheets.Add("員工資料");
    
    worksheet.Cell(1, 1).Value = "姓名";
    worksheet.Cell(1, 2).Value = "部門";
    
    // 填入資料
    var employees = employeeService.GetAll();
    int row = 2;
    foreach (var emp in employees)
    {
        worksheet.Cell(row, 1).Value = emp.Name;
        worksheet.Cell(row, 2).Value = emp.Department;
        row++;
    }
    
    using var stream = new MemoryStream();
    workbook.SaveAs(stream);
    
    return File(stream.ToArray(), 
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", 
        "員工資料.xlsx");
}
```

## 總結：重要觀念回顧

### Content-Type 核心概念

1. **Content-Type 是標籤**：告訴接收方資料格式
2. **請求 vs 回應**：兩個方向都可能有 Content-Type
3. **瀏覽器自動處理**：大部分情況自動設定
4. **影響解析方式**：錯誤的 Content-Type 導致解析失敗

### 三種主要表單 enctype

1. **application/x-www-form-urlencoded**：預設，一般表單，格式為 `key=value&key=value`
2. **multipart/form-data**：檔案上傳必須，使用 boundary 分隔
3. **text/plain**：很少使用，格式為 `key=value\nkey=value`

### jQuery AJAX 重要限制

1. **檔案上傳不能用 JSON**：檔案是二進位資料，JSON 只能處理文字
2. **JSON 要轉字串**：使用 `JSON.stringify()` 將物件轉成字串
3. **檔案上傳設定**：`processData: false, contentType: false`

### 瀏覽器資源載入原理

1. **CSS 載入**：解析規則，套用樣式，存在樣式表記憶體，不修改 HTML
2. **JavaScript 載入**：立即執行程式碼，存在全域作用域，可能修改 HTML
3. **原始碼不變**：資源載入不會改變 HTML 原始碼，只能在開發者工具中看到

### 最重要的實務原則

1. **有檔案上傳就用 multipart/form-data**，不能用 JSON
2. **AJAX 發送 JSON 要用 JSON.stringify()**
3. **檔案上傳的 AJAX 要設定 contentType: false**
4. **遇到問題先檢查 F12 Network 標籤**
5. **後端正確設定回應的 Content-Type**
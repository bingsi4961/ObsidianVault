---
date: 2025-07-31 14:37
aliases: 
tags: []
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
## 1. HTML Form 表單提交的 Content-Type

### 1.1 基本概念

- **enctype** 全名：**encoding type** (編碼類型)
- 用途：指定表單資料在提交到伺服器時的編碼類型

### 1.2 三種 enctype 設定

#### application/x-www-form-urlencoded (預設)

```html
<!-- 預設 enctype="application/x-www-form-urlencoded" -->
<form method="post" action="/submit">
    <input type="text" name="username" />
    <input type="text" name="email" />
    <input type="submit" value="送出" />
</form>
```

**特性**：

- 資料格式：`key=value&key=value&key=value`
- 會進行 URL 編碼（如 `@` 變成 `%40`）
- 資料在 HTTP Body 中，<mark style="background: #FFF3A3A6;">不在 URL 上</mark>
- 適用：一般表單提交

#### multipart/form-data (檔案上傳必須)

```html
<form method="post" action="/submit" enctype="multipart/form-data">
    <input type="text" name="username" />
    <input type="file" name="uploadFile" />
    <input type="submit" value="送出" />
</form>
```

**特性**：

- 資料格式：使用 boundary 分隔符
- 適用：檔案上傳或混合資料
- HTTP Body 結構：

```
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="username"

john
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="uploadFile"; filename="document.pdf"
Content-Type: application/pdf

[檔案的二進位內容]
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

#### text/plain (很少使用)

```html
<form method="post" action="/submit" enctype="text/plain">
    <input type="text" name="username" />
    <input type="submit" value="送出" />
</form>
```

**特性**：

- 資料格式：`key=value\nkey=value\nkey=value`
- 使用換行符號分隔
- 不會進行 URL 編碼
- 實際開發中很少使用

### 1.3 POST vs GET 的差異

#### POST 方法

- 資料位置：HTTP Body 中
- URL：乾淨，沒有參數
- 適用：表單提交、敏感資料

#### GET 方法

- 資料位置：URL QueryString 中
- URL：包含所有參數
- 適用：查詢、非敏感資料

## 2. jQuery AJAX 中的 Content-Type

### 2.1 不同資料類型的處理

#### 一般文字資料 (預設)

```javascript
$.ajax({
    url: '/submit',
    type: 'POST',
    data: {
        username: 'john',
        email: 'john@example.com'
    }
    // 預設 Content-Type: application/x-www-form-urlencoded
});
```

#### JSON 格式

```javascript
$.ajax({
    url: '/submit',
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify({
        username: 'john',
        email: 'john@example.com'
    })
});
```

#### 檔案上傳

```javascript
var formData = new FormData();
formData.append('username', 'john');
formData.append('uploadFile', $('#fileInput')[0].files[0]);

$.ajax({
    url: '/upload',
    type: 'POST',
    data: formData,
    processData: false,    // 重要！
    contentType: false     // 重要！讓瀏覽器自動設定
});
```

### 2.2 重要限制

- <mark style="background: #FFF3A3A6;">檔案上傳時不能使用 application/json</mark>
- 有檔案就必須用 FormData + contentType: false
- JSON 格式只能處理文字資料，無法處理二進位檔案

### 2.3 混合資料的解決方案

#### 方案一：全部使用 FormData

```javascript
var formData = new FormData();
formData.append('username', 'john');
formData.append('age', 25);
formData.append('hobbies[]', 'reading');
formData.append('hobbies[]', 'baseketball');
formData.append('uploadFile', file);
```

#### 方案二：JSON 作為字串

```javascript
var formData = new FormData();
var userData = { username: 'john', age: 25 };
formData.append('userData', JSON.stringify(userData));
formData.append('uploadFile', file);
```

## 3. 完整的 HTTP Request 結構

### 3.1 HTTP Request 組成

#### Request Line (請求行)

```
POST /submit HTTP/1.1
```

#### Request Headers (請求標頭)

```
Host: localhost:5000
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 1234
User-Agent: Mozilla/5.0...
Accept: text/html,application/xhtml+xml...
Cookie: sessionId=abc123
```

#### Empty Line (空行)

```
[空行分隔 Headers 和 Body]
```

#### Request Body (請求主體)

```
實際的資料內容
```

### 3.2 常見的重要 Headers

| Header           | 說明        | 範例                               |     |
| ---------------- | --------- | -------------------------------- | --- |
| `Host`           | 目標伺服器     | `localhost:5000`                 |     |
| `Content-Type`   | 內容類型      | `application/json`               |     |
| `Content-Length` | 內容長度      | `1234`                           |     |
| `User-Agent`     | 瀏覽器資訊     | `Chrome/91.0.4472.124`           |     |
| `Accept`         | 可接受的回應類型  | `text/html,application/json`     |     |
| `Cookie`         | Cookie 資料 | `sessionId=abc123`               |     |
| `Authorization`  | 驗證資訊      | `Bearer eyJhbGciOiJIUzI1NiIs...` |     |

## 4. 常用的 Content-Type 分類

### 4.1 主要用於請求 (Request)

#### application/x-www-form-urlencoded

- 用途：HTML 表單預設格式
- 特性：資料以 `key=value&key=value` 格式傳送
	- 如果是 POST，則存在 Request Body 中
	- 如果是 GET，則存在 Url 中

#### multipart/form-data

- 用途：檔案上傳必須使用
- 特性：使用 boundary 分隔不同欄位

#### application/json

- 用途：API 資料交換
- 特性：結構化資料，易於解析

### 4.2 主要用於回應 (Response)

#### text/html

- 用途：HTML 頁面內容
- 瀏覽器行為：解析並渲染網頁

#### text/css

- 用途：CSS 樣式表檔案
- 瀏覽器行為：解析並套用樣式規則

#### application/javascript

- 用途：JavaScript 檔案
- 瀏覽器行為：解析並執行程式碼

#### application/pdf

- 用途：PDF 檔案
- 瀏覽器行為：顯示 PDF 或提示下載

#### application/octet-stream

- 用途：二進位檔案下載
- 瀏覽器行為：<mark style="background: #FFF3A3A6;">強制</mark> 下載到本地

#### application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

- 用途：Excel 檔案 (.xlsx)
- 瀏覽器行為：下載檔案

### 4.3 兩者都會用到

#### application/json

- 請求：發送 JSON 資料給伺服器
- 回應：伺服器回傳 JSON 資料

#### application/xml

- 請求：發送 XML 資料
- 回應：回傳 XML 資料

## 5. jQuery AJAX 回應處理

### 5.1 自動解析機制

jQuery 會根據以下順序決定如何解析回應：

1. 明確指定的 `dataType` (最高優先級)
2. 伺服器回應的 `Content-Type`
3. 內容的格式特徵 (智慧判斷)

### 5.2 不同 Content-Type 的處理

#### application/json

```javascript
$.ajax({
    url: '/api/getData',
    success: function(response) {
        // response 已經是 JavaScript 物件
        console.log(typeof response); // "object"
        console.log(response.username); // 直接存取屬性
    }
});
```

#### text/html

```javascript
$.ajax({
    url: '/getHtmlFragment',
    success: function(response) {
        // response 是 HTML 字串，不是 DOM 物件
        console.log(typeof response); // "string"
        $('#container').html(response); // jQuery 內部會解析並插入
    }
});
```

#### text/plain

```javascript
$.ajax({
    url: '/getPlainText',
    success: function(response) {
        // response 是純文字字串
        console.log(typeof response); // "string"
    }
});
```

### 5.3 強制指定 dataType

```javascript
$.ajax({
    url: '/api/test',
    dataType: 'json', // 強制當 JSON 處理
    success: function(response) {
        // 即使伺服器 Content-Type 錯誤，也會嘗試解析成 JSON
    }
});
```

## 6. 瀏覽器資源載入流程

### 6.1 CSS 載入流程

#### HTML 中的引用

```html
<link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css">
```

#### 完整流程

1. **瀏覽器解析 HTML**：讀到 `<link>` 標籤
2. **發送請求**：`GET /css/bootstrap.min.css`
3. **伺服器回應**：`Content-Type: text/css` + CSS 程式碼
4. **瀏覽器處理**：
    - 將 CSS 規則存在樣式表記憶體中
    - 套用到對應的 HTML 元素
    - **不會**修改 HTML 原始碼

#### 查看方式

- **F12 → Network**：看到載入的 CSS 檔案
- **F12 → Sources**：可以查看 CSS 原始碼
- **F12 → Elements → 計算**：查看計算後的最終樣式
- **檢視原始碼**：仍然只有 `<link>` 標籤

### 6.2 JavaScript 載入流程

#### HTML 中的引用

```html
<script src="/js/app.js"></script>
```

#### 完整流程

1. **瀏覽器解析 HTML**：讀到 `<script>` 標籤
2. **發送請求**：`GET /js/app.js`
3. **伺服器回應**：`Content-Type: application/javascript` + JS 程式碼
4. **瀏覽器處理**：
    - **立即執行** JavaScript 程式碼
    - 執行結果存在全域作用域中
    - **不會**修改 HTML 原始碼

#### 與 CSS 的差異

|特性|CSS|JavaScript|
|---|---|---|
|**處理方式**|套用樣式規則|執行程式碼|
|**存在位置**|樣式表記憶體|全域作用域|
|**HTML 變化**|不改變 HTML|可能改變 HTML|
|**執行時機**|載入後持續作用|載入時執行一次|

## 7. .NET Core 後端處理

### 7.1 接收不同格式的資料

#### form-urlencoded 格式

```csharp
[HttpPost]
public IActionResult Submit(string username, string email, int age)
{
    // Model Binding 會自動解析 form-urlencoded 格式
    return Json(new { success = true });
}
```

#### JSON 格式

```csharp
public class UserModel
{
    public string Username { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
}

[HttpPost]
public IActionResult SubmitJson([FromBody] UserModel model)
{
    // [FromBody] 告訴 .NET Core 從 JSON 解析
    return Json(new { success = true });
}
```

#### 檔案上傳

```csharp
[HttpPost]
public IActionResult Upload(string username, string email, IFormFile uploadFile)
{
    if (uploadFile != null && uploadFile.Length > 0)
    {
        var fileName = uploadFile.FileName;
        var contentType = uploadFile.ContentType;
        var fileSize = uploadFile.Length;
    }
    return Json(new { success = true });
}
```

#### text/plain 格式

```csharp
[HttpPost]
public async Task<IActionResult> Submit()
{
    // 需要手動讀取 Body
    using var reader = new StreamReader(Request.Body);
    string body = await reader.ReadToEndAsync();
    
    // 手動解析格式：username=john\nemail=test@asus.com
    var lines = body.Split('\n');
    foreach (var line in lines)
    {
        var parts = line.Split('=');
        if (parts.Length == 2)
        {
            Console.WriteLine($"{parts[0]}: {parts[1]}");
        }
    }
    
    return View();
}
```

### 7.2 設定回應的 Content-Type

#### HTML 回應

```csharp
public IActionResult Index()
{
    return View(); // 自動設定 Content-Type: text/html
}
```

#### JSON 回應

```csharp
public IActionResult GetData()
{
    var data = new { username = "john", email = "test@asus.com" };
    return Json(data); // 自動設定 Content-Type: application/json
}
```

#### 檔案下載

```csharp
public IActionResult DownloadExcel()
{
    // 使用 ClosedXML 產生 Excel
    var excelBytes = GenerateExcelFile();
    return File(excelBytes, 
        "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", 
        "report.xlsx");
}

public IActionResult DownloadAnyFile()
{
    var fileBytes = System.IO.File.ReadAllBytes("data.bin");
    return File(fileBytes, "application/octet-stream", "data.bin");
}
```

#### 自訂 Content-Type

```csharp
public IActionResult GetCustomCss()
{
    var css = "body { background-color: red; }";
    return Content(css, "text/css");
}

public IActionResult GetCustomJs()
{
    var js = "console.log('Hello from server');";
    return Content(js, "application/javascript");
}
```

## 8. 開發者工具的使用

### 8.1 F12 開發者工具各標籤

#### Network 標籤

- 用途：查看所有 HTTP 請求和回應
- 可看到：Request Headers、Response Headers、Status Code、Content-Type

#### Sources 標籤

- 用途：查看載入的資源原始碼
- 可看到：CSS 檔案內容、JavaScript 檔案內容

#### Elements 標籤

- **計算 (Computed)** 標籤：查看元素的最終計算樣式
- **樣式 (Styles)** 標籤：查看所有相關的 CSS 規則

#### Console 標籤

- 用途：查看 JavaScript 執行結果、錯誤訊息
- 可執行：JavaScript 程式碼測試

### 8.2 實用的檢查方法

#### 檢查請求內容

```javascript
// 在 Controller 中記錄請求資訊
foreach (var header in Request.Headers)
{
    Console.WriteLine($"{header.Key}: {header.Value}");
}
Console.WriteLine($"Content-Type: {Request.ContentType}");
Console.WriteLine($"Content-Length: {Request.ContentLength}");
```

#### 檢查載入的資源

```javascript
// 在瀏覽器 Console 中執行
console.log('載入的樣式表:', document.styleSheets);
console.log('載入的腳本:', document.scripts);

// 檢查計算樣式
var element = document.querySelector('.test-box');
var computedStyle = window.getComputedStyle(element);
console.log('計算後的顏色:', computedStyle.color);
```

## 9. 實際開發應用 (Portal & GTS 系統)

### 9.1 Portal 系統 (.NET Core 3.1)

#### 常用的前端技術棧

- **Bootstrap 4.3.1**：UI 框架
- **jQuery 3.3.1**：JavaScript 函式庫
- **ClosedXML 0.95.2**：Excel 處理

#### 典型的 AJAX 應用

```javascript
// 資料提交
$.ajax({
    url: '/api/saveData',
    type: 'POST',
    contentType: 'application/json',
    data: JSON.stringify({
        employeeId: 'A123456',
        department: 'IT'
    }),
    success: function(response) {
        if (response.success) {
            alert('儲存成功');
        }
    }
});

// 檔案上傳
var formData = new FormData();
formData.append('file', $('#fileInput')[0].files[0]);
$.ajax({
    url: '/api/upload',
    type: 'POST',
    data: formData,
    processData: false,
    contentType: false,
    success: function(response) {
        console.log('上傳成功');
    }
});
```

### 9.2 GTS 系統 (.NET Framework 4.8)

#### 常用的前端技術棧

- **Bootstrap 3.0.0**：UI 框架
- **jQuery 1.10.2**：JavaScript 函式庫
- **ClosedXML 0.95.4**：Excel 處理

#### 相容性考量

- 較舊的 jQuery 版本語法
- Bootstrap 3.x 的 CSS 類別名稱差異

## 10. 最佳實務建議

### 10.1 Content-Type 設定原則

#### 前端原則

- **一般表單**：使用預設的 `application/x-www-form-urlencoded`
- **檔案上傳**：必須使用 `multipart/form-data`
- **API 呼叫**：使用 `application/json`
- **檔案上傳 + 資料**：使用 FormData，不要用 JSON

#### 後端原則

- **網頁回應**：確保設定 `text/html`
- **API 回應**：確保設定 `application/json`
- **檔案下載**：正確設定檔案的 MIME Type
- **強制下載**：使用 `application/octet-stream`

### 10.2 除錯技巧

#### 前端除錯

- 使用 F12 Network 標籤檢查請求
- 檢查 Content-Type 是否正確
- 使用 Console 輸出回應內容類型

#### 後端除錯

- 記錄接收到的 Headers
- 檢查 Model Binding 是否正確
- 確認回應的 Content-Type 設定

### 10.3 常見錯誤避免

#### 檔案上傳錯誤

- ❌ 忘記設定 `enctype="multipart/form-data"`
- ❌ AJAX 中使用 `contentType: 'application/json'` 上傳檔案
- ✅ 使用 FormData + `contentType: false`

#### Content-Type 錯誤

- ❌ 後端回傳 JSON 但設定為 `text/html`
- ❌ 前端發送 JSON 但設定為 `text/plain`
- ✅ 確保前後端 Content-Type 一致

#### jQuery 解析錯誤

- ❌ 依賴自動解析，但伺服器 Content-Type 錯誤
- ✅ 適當時候明確指定 `dataType`
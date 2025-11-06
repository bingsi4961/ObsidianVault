---
date: 2025-11-06 14:37
aliases:
tags:
  - JavaScript
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[$.param()]]

---
## 一、AJAX 成功後重新載入頁面

### 保留所有參數重新載入

```javascript
// 重新載入頁面（保留所有參數）
window.location.reload();
```

**說明：** `window.location.reload()` 會保留目前 URL 的所有內容（路徑、QueryString、Hash(#)）

### 導向指定 URL 並帶參數

```javascript
// 方法 1: 導向指定 URL 並帶參數
var params = {
    id: 123,
    type: 'edit'
};
var queryString = $.param(params);
window.location.href = '@Url.Action("Index")' + '?' + queryString;
```

---
## 二、window.location 屬性差異

假設目前 URL：`https://example.com/Product/List?category=A&page=2#section1`

| 屬性                         | 回傳值                                                           | 說明                 |
| -------------------------- | ------------------------------------------------------------- | ------------------ |
| `window.location.href`     | `https://example.com/Product/List?category=A&page=2#section1` | 完整 URL             |
| `window.location.pathname` | `/Product/List`                                               | 路徑（不含網域、參數、hash）   |
| `window.location.search`   | `?category=A&page=2`                                          | QueryString（含 `?`） |
| `window.location.hash`     | `#section1`                                                   | Hash（含 `#`）        |
| `window.location.hostname` | `example.com`                                                 | 網域                 |

### 使用情境

- **href**：取得或設定完整 URL，常用於導向其他頁面
- **pathname**：取得路徑，常用於判斷目前在哪個頁面
- **search**：取得參數字串，常用於解析 QueryString

---
## 三、URLSearchParams 常用方法

### 建立物件

```javascript
var urlParams = new URLSearchParams(window.location.search);
// 假設 window.location.search = "?category=A&page=2&status=active"
```

### 1. get(key) - 取得參數值

```javascript
var category = urlParams.get('category');  // "A"
var page = urlParams.get('page');          // "2"
var notExist = urlParams.get('name');      // null（不存在的參數）
```

### 2. set(key, value) - 設定參數值（會覆蓋）

```javascript
// 修改現有參數
urlParams.set('page', '3');     // page 從 2 改為 3

// 新增不存在的參數
urlParams.set('sort', 'desc');  // 新增 sort=desc

// 結果: category=A&page=3&status=active&sort=desc
```

### 3. append(key, value) - 附加參數（不會覆蓋）

```javascript
// 假設原本有 tag=old
urlParams.append('tag', 'new');  // 不會覆蓋，會並存

// 結果: tag=old&tag=new （同名參數可以有多個值）
```

**set 與 append 的差異：**

- **set**：如果參數已存在，會**覆蓋**舊值；如果不存在，會新增
- **append**：無論參數是否存在，都會**附加**新值，允許同名參數有多個值

```javascript
// 範例比較
var params = new URLSearchParams('tag=old');

// 使用 set
params.set('tag', 'new');
console.log(params.toString());  // 結果: tag=new （覆蓋）

// 使用 append
params.append('tag', 'new');
console.log(params.toString());  // 結果: tag=old&tag=new （並存）
```

### 4. toString() - 轉換為字串

```javascript
var queryString = urlParams.toString();
// 結果: "category=A&page=3&status=active&sort=desc"
// 注意：沒有前面的 "?" 符號

// 通常這樣使用：
window.location.href = window.location.pathname + '?' + urlParams.toString();
```

### 5. 其他常用方法

```javascript
urlParams.delete('page');          // 刪除參數
urlParams.has('category');         // 檢查參數是否存在（回傳 true/false）
urlParams.getAll('tag');           // 取得所有同名參數的值（陣列）

// 遍歷所有參數
urlParams.forEach(function(value, key) {
    console.log(key + ' = ' + value);
});
```

---
## 四、實際應用範例

### 範例 1：修改頁碼並重新載入

```javascript
var urlParams = new URLSearchParams(window.location.search);
urlParams.set('page', '1');  // 設定為第一頁
window.location.href = window.location.pathname + '?' + urlParams.toString();
```

### 範例 2：檢查是否有特定參數

```javascript
var urlParams = new URLSearchParams(window.location.search);
if (urlParams.get('category') === 'A') {
    console.log('目前在 A 類別');
}
```

### 範例 3：刪除參數後重新載入

```javascript
var urlParams = new URLSearchParams(window.location.search);
urlParams.delete('status');  // 移除 status 參數
window.location.href = window.location.pathname + '?' + urlParams.toString();
```

### 範例 4：清除所有參數重新載入

```javascript
window.location.href = window.location.pathname;
// 🚨/Product/List 可以正常瀏覽
```

### 範例 5：手動加入參數

```javascript
var queryObject = { name: 'John', age: 44, addr: 'taipei' }; 
var urlParams = new URLSearchParams(queryObject);

// ----------------------------------------------------------
var urlParams = new URLSearchParams(); 
urlParams.set('name', 'John'); 
urlParams.set('age', 44); 
urlParams.set('addr', 'taipei');

console.log(urlParams.toString());
// name=John&age=44&addr=taipei
```

---
## 五、快速參考表

|需求|語法|
|---|---|
|保留參數重新載入|`window.location.reload()`|
|清除參數重新載入|`window.location.href = window.location.pathname`|
|取得參數值|`urlParams.get('key')`|
|設定參數值（覆蓋）|`urlParams.set('key', 'value')`|
|附加參數（不覆蓋）|`urlParams.append('key', 'value')`|
|刪除參數|`urlParams.delete('key')`|
|檢查參數是否存在|`urlParams.has('key')`|
|取得所有同名參數|`urlParams.getAll('key')`|
|轉換為 QueryString|`urlParams.toString()`|

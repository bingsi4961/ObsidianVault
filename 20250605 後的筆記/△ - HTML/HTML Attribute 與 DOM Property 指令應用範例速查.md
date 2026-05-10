---
date: 2026-05-04 15:30
title:
aliases:
tags:
  - HTML
  - JavaScript
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
#### 📑 [[HTML Attribute 與 DOM Property 完整教學指南]]
#### 📑 [[HTML Attribute 與 DOM Property 完全指南（一）：搞懂藍圖與房子的差異]]
#### 📑 [[HTML Attribute 與 DOM Property 完全指南（二）：各元件對應關係完整拆解]]
#### 📑 [[HTML Attribute 與 DOM Property 完全指南（三）：實戰備忘錄與速查表]]

---

## 1. 文字輸入框（Input Text）

```html
<input type="text" id="username" value="初始值">
```

### 原生 JavaScript

```javascript
const input = document.getElementById('username');

// 讀取
input.getAttribute('value')  // "初始值"（HTML attribute，永遠是初始值）
input.value                  // 使用者當前輸入的值（DOM property，即時狀態）

// 設定新值
input.value = '新值';          // 畫面立即更新（改 DOM property）
input.getAttribute('value')  // 仍然是 "初始值"（attribute 沒有跟著改）
```

### jQuery

```javascript
// 讀取
$('#username').attr('value')  // "初始值"（HTML attribute，永遠是初始值）
$('#username').prop('value')  // 使用者當前輸入的值
$('#username').val()          // 同上，推薦使用這個，語意最清楚

// 設定新值
$('#username').val('新值');          // ✅ 推薦：畫面立即更新
$('#username').prop('value', '新值') // 也可以，但 val() 更語意化
$('#username').attr('value')         // 設定之後仍然是 "初始值"

// 清空欄位
$('#username').val('');                    // 單純清空畫面（一般情境用這個）
$('#username').val('').attr('value', ''); // 防禦性清空：同時抹除 attribute，
                                          // 防止 form.reset() 時舊值回魂
```

---

## 2. 核取方塊（Checkbox）

```html
<input type="checkbox" id="terms" checked>
```

### 原生 JavaScript

```javascript
const terms = document.getElementById('terms');

// 讀取
terms.getAttribute('checked')  // "checked"（attribute 回傳字串）
terms.checked                  // true（DOM property 回傳布林值）

// 改變狀態（取消勾選）
terms.checked = false;
terms.getAttribute('checked')  // 仍然是 "checked"（attribute 不跟著改）
```

### jQuery

```javascript
// 讀取
$('#terms').attr('checked')  // "checked"（字串）
$('#terms').prop('checked')  // true（布林值）

// 改變狀態
$('#terms').prop('checked', false);  // ✅ 正確：取消勾選
$('#terms').prop('checked', true);   // ✅ 正確：強制打勾

// ❌ 錯誤示範：不要用 attr 改勾選狀態
// $( '#terms').attr('checked', true);
// → 在舊版 jQuery 或某些情境下，畫面框框根本不會被打勾！
// → attr 只是改了 HTML 標籤上的字，沒有真正觸發狀態改變。

// 改變後，attribute 依然不動
$('#terms').attr('checked')  // 仍然是 "checked"
```

> ⚠️ **通用原則**：`checked`、`disabled`、`readonly` 這類布林值狀態，一律用 `.prop()` 操作，不要用 `.attr()`。

---

## 3. 下拉選單（Select）

```html
<select id="citySelect">
    <option value="taipei" selected>台北</option>
    <option value="tainan">台南</option>
</select>
```

### 原生 JavaScript

```javascript
const select = document.getElementById('citySelect');

// 讀取
select.getAttribute('value')  // null
                              // ⚠️ 注意：<select> 元素本身沒有 value attribute。
                              // 它的當前選中值是由 <option> 的 selected 狀態決定的，
                              // 而不是 <select> 本身的屬性，所以這裡回傳 null。
select.value                  // "taipei"（DOM property，回傳當前選中的 option 值）

// 改變選中值
select.value = 'tainan';      // 程式切換到台南
```

### jQuery

```javascript
// 讀取 select 的當前選中值
$('#citySelect').attr('selected')  // undefined（<select> 本身沒有此 attribute）
$('#citySelect').prop('selected')  // undefined（同上）
$('#citySelect').val()             // "taipei" ✅ 這是正確的取值方式

// 讀取個別 option 的狀態（通常不需要，但偶爾會用到）
$('#citySelect option:first').attr('selected')  // "selected"（字串）
$('#citySelect option:first').prop('selected')  // true（布林值）

// 改變選中值
$('#citySelect').val('tainan');  // ✅ 推薦：直接給值，jQuery 自動切換對應的 option
                                 // 不需要自己去找 option 再設定 prop('selected', true)
```

---

## 4. 停用狀態（Disabled）

```html
<input type="text" id="myInput" value="原始文字" disabled>
```

### 原生 JavaScript

```javascript
const input = document.getElementById('myInput');

// 讀取
input.getAttribute('disabled')  // ""（空字串，HTML attribute 的布林值寫法）
input.disabled                  // true（DOM property 回傳布林值）

// 啟用輸入框（改變 DOM property）
input.disabled = false;
input.getAttribute('disabled')  // 仍然是 ""（attribute 不跟著同步）
```

### jQuery

```javascript
// 讀取
$('#myInput').attr('disabled')  // "disabled"（字串）
$('#myInput').prop('disabled')  // true（布林值）

// 啟用 / 停用（用 prop，不要用 attr）
$('#myInput').prop('disabled', false);  // ✅ 啟用
$('#myInput').prop('disabled', true);   // ✅ 停用

// 改變後，attribute 不跟著動
$('#myInput').attr('disabled')  // 啟用後仍然可能顯示 "disabled"，
                                // 因為 attribute 不會因 prop 的改變而更新
```

---

## 5. 必填驗證（Required）

```html
<form id="myForm">
    <input type="text" id="username" value="預設用戶名" required>
</form>
```

### 原生 JavaScript

```javascript
const username = document.getElementById('username');

// 讀取
username.getAttribute('required')  // ""（空字串，存在即代表必填）
username.required                  // true（DOM property 回傳布林值）

// 動態移除必填驗證
username.required = false;
username.getAttribute('required')  // 仍然存在（attribute 沒有跟著移除）
```

### jQuery

```javascript
// 讀取
$('#username').attr('required')  // ""
$('#username').prop('required')  // true

// 動態移除必填驗證
$('#username').prop('required', false);  // ✅ 正確方式
```

---

## 6. 自訂資料屬性（data-*）

```html
<div id="user-card"
     data-user-id="123"
     data-role="admin"
     data-last-login="2024-01-01">
</div>
```

### 原生 JavaScript（getAttribute vs dataset）

```javascript
const userCard = document.getElementById('user-card');

// 使用 getAttribute（回傳字串）
userCard.getAttribute('data-user-id')    // "123"（字串）
userCard.getAttribute('data-role')       // "admin"（字串）
userCard.getAttribute('data-last-login') // "2024-01-01"（字串）

// 使用 dataset API（自動轉型，命名自動轉為 camelCase）
userCard.dataset.userId    // "123"（⚠️ 原生 dataset 不自動轉型，仍是字串）
userCard.dataset.role      // "admin"
userCard.dataset.lastLogin // "2024-01-01"
// 注意：HTML 的 data-user-id（kebab-case）對應到 dataset.userId（camelCase）

// 使用 dataset 修改（這一點和 jQuery .data() 行為不同！）
userCard.dataset.lastLogin = '2024-01-02';
userCard.getAttribute('data-last-login')  // "2024-01-02"
// ✅ 原生 dataset 的修改「會直接反映到 HTML 標籤」
```

### jQuery（attr vs data）

```javascript
// 使用 attr（回傳字串）
$('#user-card').attr('data-user-id')    // "123"（字串）
$('#user-card').attr('data-role')       // "admin"（字串）

// 使用 .data()（自動型別轉換，語法更簡潔）
$('#user-card').data('userId')    // 123（數字）← jQuery 自動轉型
$('#user-card').data('role')      // "admin"（字串）
$('#user-card').data('lastLogin') // "2024-01-01"（字串）
// 注意：.data() 也使用 camelCase 命名

// 使用 .data() 修改（⚠️ 超級大坑！）
$('#user-card').data('role', 'guest');       // 只改了 jQuery 記憶體
$('#user-card').attr('data-role')            // 仍然是 "admin"！HTML 根本沒變！

// 如果需要讓修改反映到 HTML 標籤（例如搭配 CSS 屬性選擇器），要改用 attr
$('#user-card').attr('data-role', 'guest');  // ✅ 這樣 HTML 才會真的更新
$('#user-card').attr('data-role')            // "guest"，這樣才有變
```

> 💣 **重要對比**： 原生 `dataset` 修改 → ✅ **會**更新到 HTML 標籤 jQuery `.data()` 修改 → ❌ **不會**更新到 HTML 標籤（只改記憶體） 這是兩者最大的行為差異，務必記住。

---

## 7. Class 樣式操作

```html
<div id="box" class="container blue"></div>
```

### 原生 JavaScript（className vs classList）

```javascript
const box = document.getElementById('box');

// 讀取
box.getAttribute('class')      // "container blue"（字串）
box.className                  // "container blue"（字串，DOM property）
box.classList.contains('blue') // true

// 使用 classList API 精準操作（推薦）
box.classList.add('large');    // 只新增 large，container 和 blue 不受影響
box.classList.remove('blue');  // 只移除 blue
box.classList.toggle('visible'); // 有就移除，沒有就新增

// ✅ classList 的操作會同時反映在 attribute 和 className
box.getAttribute('class') // "container large visible"
box.className             // "container large visible"

// ❌ 不要直接改 className 字串（會全部覆蓋）
// box.className = 'large'; → container 和 blue 都會消失
```

### jQuery

```javascript
// 讀取
$('#box').attr('class')       // "container blue"
$('#box').prop('className')   // "container blue"

// ❌ 不要用 attr 修改 class（會整個覆蓋！）
// $('#box').attr('class', 'large');
// → 結果：class="large"，container 和 blue 都不見了

// ✅ 正確方式：使用專用方法精準操作
$('#box').addClass('large');      // 新增，其他 class 不受影響 → "container blue large"
$('#box').removeClass('blue');    // 移除指定的 → "container large"
$('#box').toggleClass('visible'); // 智慧切換（有就脫，沒有就穿）

// jQuery 的 addClass/removeClass/toggleClass 對應原生的 classList API
// 概念和行為完全一樣，只是語法不同
```

---

## 8. 行內樣式（Style）

```html
<div id="styled" style="color: red; font-size: 16px;"></div>
```

### 原生 JavaScript

```javascript
const styled = document.getElementById('styled');

// 讀取整個 style 字串
styled.getAttribute('style')  // "color: red; font-size: 16px;"

// 讀取個別樣式（DOM property，camelCase 命名）
styled.style.color      // "red"
styled.style.fontSize   // "16px"

// 動態設定樣式
styled.style.backgroundColor = 'yellow';
styled.getAttribute('style') // "color: red; font-size: 16px; background-color: yellow;"
                             // ✅ 行內 style 的修改「會同時反映到 attribute」
```

### jQuery

```javascript
// 讀取整個 style 字串
$('#styled').attr('style')  // "color: red; font-size: 16px;"

// 讀取個別樣式
$('#styled').css('color')      // "rgb(255, 0, 0)"（jQuery 會轉成 rgb 格式）
$('#styled').css('font-size')  // "16px"

// 設定樣式（⚠️ 實務上不建議：把樣式寫死在 JS 裡，日後難以維護）
$('#styled').css('background-color', 'yellow');

// ✅ 實務建議：改用切換 class 的方式控制外觀
// 在 CSS 中定義好樣式 → 在 JS 中只做 addClass/removeClass
```

---

## 9. 連結與圖片的 URL 展開特性

這是一個容易被忽略的特殊行為。`href` 和 `src` 這類屬性，在 DOM property 層級會被瀏覽器自動展開成完整的絕對 URL。

### 連結（Anchor）

```html
<a href="/path" id="myLink">連結</a>
```

```javascript
const link = document.getElementById('myLink');

// HTML attribute：回傳你寫的原始值
link.getAttribute('href')  // "/path"

// DOM property：回傳瀏覽器展開的完整 URL
link.href  // "https://example.com/path"
```

### 圖片（Image）

```html
<img src="./images/photo.jpg" id="myImage">
```

```javascript
const img = document.getElementById('myImage');

// HTML attribute：回傳原始相對路徑
img.getAttribute('src')  // "./images/photo.jpg"

// DOM property：回傳完整絕對 URL
img.src  // "https://example.com/images/photo.jpg"

// 動態更換圖片（改 DOM property）
img.src = './images/new-photo.jpg';
img.getAttribute('src')  // 仍然是 "./images/photo.jpg"（attribute 不跟著改）
```

### jQuery

```javascript
// jQuery 的 .attr() 對應原生的 getAttribute，回傳原始值
$('#myLink').attr('href')  // "/path"

// jQuery 的 .prop() 對應原生的 DOM property，回傳展開的完整 URL
$('#myLink').prop('href')  // "https://example.com/path"
```

> ⚠️ **實務提醒**：如果你要抓取 `href` 的值去做字串比對（例如判斷是否為某個路徑），請用 `.attr('href')`，而不是 `.prop('href')`，否則你拿到的是完整 URL，字串比對會失敗。

---

## 10. 名稱對照差異（class / className）

有些 HTML attribute 的名稱和對應的 DOM property 名稱不一樣，這是瀏覽器的歷史遺留設計。最常見的就是 `class`：

|HTML Attribute 名稱|DOM Property 名稱|說明|
|:--|:--|:--|
|`class`|`className`|`class` 是 JavaScript 保留字，所以 DOM 改用 `className`|
|`for`（label 用）|`htmlFor`|`for` 也是 JS 保留字|
|`data-user-id`|`dataset.userId`|kebab-case 自動轉換為 camelCase|

```html
<div id="example" class="container blue"></div>
```

```javascript
const el = document.getElementById('example');

// HTML attribute（用原本的名稱 "class"）
el.getAttribute('class')  // "container blue"

// DOM property（用 JavaScript 相容的名稱 "className"）
el.className  // "container blue"
```

```javascript
// jQuery 也遵循同樣規則
$('#example').attr('class')       // "container blue"（用 "class"）
$('#example').prop('className')   // "container blue"（用 "className"）
```

---

## 附錄：原生 JS vs jQuery 語法總對照表

| 操作目的               | 原生 JavaScript                       | jQuery                       |
| :----------------- | :---------------------------------- | :--------------------------- |
| 讀取 HTML attribute  | `element.getAttribute('attr')`      | `$el.attr('attr')`           |
| 寫入 HTML attribute  | `element.setAttribute('attr', val)` | `$el.attr('attr', val)`      |
| 讀取 DOM property    | `element.propertyName`              | `$el.prop('propName')`       |
| 寫入 DOM property    | `element.propertyName = val`        | `$el.prop('propName', val)`  |
| 讀寫表單值              | `element.value`                     | `$el.val()` / `$el.val(val)` |
| 讀取 data-*          | `element.dataset.key`               | `$el.data('key')`            |
| 修改 data-*（同步 HTML） | `element.dataset.key = val`         | `$el.attr('data-key', val)`  |
| 修改 data-*（只改記憶體）   | ─                                   | `$el.data('key', val)`       |
| 新增 class           | `element.classList.add('c')`        | `$el.addClass('c')`          |
| 移除 class           | `element.classList.remove('c')`     | `$el.removeClass('c')`       |
| 切換 class           | `element.classList.toggle('c')`     | `$el.toggleClass('c')`       |
| 設定行內樣式             | `element.style.prop = val`          | `$el.css('prop', val)`       |

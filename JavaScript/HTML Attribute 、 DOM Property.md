---
title: HTML Attribute 、 DOM Property
tags: [JavaScript, HTML]

---

# Start

1. HTML 屬性（Attributes）:
- 定義在 HTML 標記中的原始值
- 通過 getAttribute() 和 setAttribute() 訪問和修改
- 值永遠是字串
- 反映 HTML 的初始狀態，不會隨著用戶互動而改變
```html
<input type="text" value="初始值">
```

2. DOM 屬性（Properties）:
- 存在於 DOM 物件中
- 直接通過點運算符訪問
- 可以是任何 JavaScript 類型（字串、布林、數字等）
- 會動態反映元素的當前狀態
```javascript
// DOM property
element.value = "新的值";  
element.checked = true;
```

主要差異舉例：

1. 值的同步
- 某些 property 的改變不會反映到 attribute（如 input 的 value）
- 某些 property 會自動同步（如 id, class）

```html
<input type="text" value="初始值">
```
```javascript
// HTML attribute 保持不變
input.getAttribute('value') // 返回 "初始值"

// DOM property 會反映當前值
input.value // 返回用戶輸入的當前值
```

2. 名稱的差異
```html
<div class="example"></div>
```
```javascript
// HTML attribute
element.getAttribute('class') // "example"

// DOM property
element.className // "example"
```

3. 布林值的處理
```html
<input type="checkbox" checked>
```
```javascript
// HTML attribute
input.getAttribute('checked') // 返回空字串或 "checked"

// DOM property
input.checked // 返回 true 或 false
```

4. 自定義屬性：
```html
<div data-custom="123"></div>
```
```javascript
// HTML attribute
div.getAttribute('data-custom') // "123"

// DOM property
div.dataset.custom // "123"
```

一些實用提示：

1. 當需要：
- 讀取初始值時 → 使用 getAttribute()
- 獲取當前狀態時 → 使用 DOM property
 - 設置動態值時 → 使用 DOM property

2. jQuery 的處理方式：
- .attr() 方法：操作 HTML attribute
- .prop() 方法：操作 DOM property
- .val() 方法：智能處理表單元素的值

3. 使用 .prop() 處理：
- checked, selected, disabled, readonly 等布林值屬性
- 表單元素的當前狀態

4. 使用 .attr() 處理：
- HTML 自定義屬性
- 需要獲取初始值的情況
- data-* 屬性（雖然也可以用 .data()）

5. 使用 .val() 處理：
- 所有表單元素的值

6. 特殊情況：
- `href` 屬性：
```html
<a href="/path">連結</a>
```
```javascript
// HTML attribute 返回原始值
link.getAttribute('href') // "/path"

// DOM property 返回完整 URL
link.href // "https://example.com/path"
```

要注意的是，大多數情況下，HTML attributes 和 DOM properties 會自動保持同步，但有些特殊情況（如 input 的 value、checkbox 的 checked）會有上述的差異。了解這些差異對於處理表單元素和用戶交互特別重要。

# 更多具體範例來深入說明

1. Select 元素的選項值：
```html
<select id="citySelect">
    <option value="taipei" selected>台北</option>
    <option value="tainan">台南</option>
</select>
```
```javascript
const select = document.getElementById('citySelect');

// HTML attribute
select.getAttribute('value') // null，因為 select 本身沒有 value attribute

// DOM property
select.value // "taipei"，返回當前選中的選項值
```

2. Input 輸入框的狀態：
```html
<input type="text" value="原始文字" id="myInput" disabled>
```
```javascript
const input = document.getElementById('myInput');

// disabled 狀態
input.getAttribute('disabled') // 返回空字串
input.disabled // 返回 true

// 啟用輸入框
input.disabled = false; // DOM property 改變
input.getAttribute('disabled') // 仍然返回空字串，HTML attribute 不變
```

3. Image 圖片的路徑：
```html
<img src="./images/photo.jpg" id="myImage">
```
```javascript
const img = document.getElementById('myImage');

// HTML attribute
img.getAttribute('src') // "./images/photo.jpg"

// DOM property
img.src // "https://example.com/images/photo.jpg"（完整 URL）

// 動態改變圖片
img.src = "./images/new-photo.jpg" // DOM property 更新
img.getAttribute('src') // 仍然是 "./images/photo.jpg"
```

4. 表單元素的狀態管理：
```html
<form id="myForm">
    <input type="checkbox" id="terms" checked>
    <input type="text" id="username" value="預設用戶名" required>
</form>
```
```javascript
const terms = document.getElementById('terms');
const username = document.getElementById('username');

// Checkbox 狀態
terms.getAttribute('checked') // "checked" (對應 HTML attribute)
terms.checked // true (對應 DOM property)

terms.checked = false; // 更改狀態
terms.getAttribute('checked') // 仍然是 "checked"

// Required 驗證
username.getAttribute('required') // ""
username.required // true
username.required = false; // 移除必填驗證
username.getAttribute('required') // 仍然存在
```

5. 自定義資料屬性的操作：
```html
<div id="user-card" 
     data-user-id="123" 
     data-role="admin" 
     data-last-login="2024-01-01">
</div>
```
```javascript
const userCard = document.getElementById('user-card');

// 使用 getAttribute
userCard.getAttribute('data-user-id') // "123"
userCard.getAttribute('data-role') // "admin"

// 使用 dataset API（DOM property）
userCard.dataset.userId // "123"
userCard.dataset.role // "admin"

// 動態更新資料
userCard.dataset.lastLogin = "2024-01-02"
userCard.getAttribute('data-last-login') // 會更新為 "2024-01-02"
```

6. Class 操作的差異：
```html
<div id="box" class="container blue">
```
```javascript
const box = document.getElementById('box');

// HTML attribute
box.getAttribute('class') // "container blue"

// DOM properties
box.className // "container blue"
box.classList.contains('blue') // true

// 使用 classList API 操作（推薦方式）
box.classList.add('large');
box.classList.remove('blue');
box.classList.toggle('visible');

// 結果反映在兩者
box.getAttribute('class') // "container large visible"
box.className // "container large visible"
```

7. 樣式操作：
```html
<div id="styled" style="color: red; font-size: 16px;">
```
```javascript
const styled = document.getElementById('styled');

// HTML attribute
styled.getAttribute('style') // "color: red; font-size: 16px;"

// DOM property
styled.style.color // "red"
styled.style.fontSize // "16px"

// 動態設置樣式
styled.style.backgroundColor = 'yellow';
styled.getAttribute('style') // 包含新增的樣式
```
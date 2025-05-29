---
title: attr()、prop() 的使用
tags: [jQuery, 技術文件]

---

1. 基本對照：
```javascript
// HTML element
<input type="checkbox" id="checkbox" checked>

// 原生 JavaScript
element.getAttribute('checked')  // 對應 HTML attribute
element.checked                 // 對應 DOM property

// jQuery
$('#checkbox').attr('checked')  // 對應 HTML attribute
$('#checkbox').prop('checked')  // 對應 DOM property
```

2. 複選框範例：
```javascript
// HTML
<input type="checkbox" id="terms" checked>

// jQuery 方式
$('#terms').attr('checked')  // "checked"
$('#terms').prop('checked')  // true

// 改變狀態
$('#terms').prop('checked', false);  // 更改 checkbox 狀態
$('#terms').attr('checked')  // 仍然返回 "checked"
```

3. 表單元素值的操作：
```javascript
// HTML
<input type="text" id="username" value="初始值">

// jQuery 操作
$('#username').attr('value')  // "初始值"

// 獲取當前值
$('#username').prop('value')  // 返回當前輸入的值
$('#username').val()         // 推薦使用 val() 方法

// 設置新值
$('#username').prop('value', '新值');
$('#username').val('新值');    // 推薦使用

$('#username').attr('value')  // 仍然返回 "初始值"
```

4. Select 元素操作：
```javascript
// HTML
<select id="citySelect">
    <option value="taipei" selected>台北</option>
    <option value="tainan">台南</option>
</select>

// jQuery 操作
$('#citySelect').attr('selected')  // undefined
$('#citySelect').prop('selected')  // undefined
$('#citySelect').val()            // "taipei"

// 選項的 selected 狀態
$('#citySelect option:first').attr('selected')  // "selected"
$('#citySelect option:first').prop('selected')  // true

// 改變選中值
$('#citySelect').val('tainan');  // 改變選中項
```

5. Disabled 狀態操作：
```javascript
// HTML
<input type="text" id="input" disabled>

// jQuery 操作
$('#input').attr('disabled')  // "disabled"
$('#input').prop('disabled')  // true

// 啟用/禁用
$('#input').prop('disabled', false);  // 啟用
$('#input').prop('disabled', true);   // 禁用
```

6. 自定義資料屬性：
```javascript
// HTML
<div id="user" data-id="123" data-role="admin">

// jQuery 操作
// 使用 attr
$('#user').attr('data-id')    // "123"
$('#user').attr('data-role')  // "admin"

// 使用 data 方法（推薦）
$('#user').data('id')         // "123"
$('#user').data('role')       // "admin"
```

7. Class 操作：
```javascript
// HTML
<div id="box" class="container blue">

// jQuery 操作
$('#box').attr('class')      // "container blue"
$('#box').prop('className')  // "container blue"

// 推薦使用的 class 操作方法
$('#box').addClass('large');
$('#box').removeClass('blue');
$('#box').toggleClass('visible');
```

使用建議：

1. 什麼時候使用 attr()：
- 需要讀取或設置 HTML 屬性的初始值
- 處理自定義資料屬性（雖然 data() 更推薦）
- 需要完全移除屬性時（使用 removeAttr()）

2. 什麼時候使用 prop()：
- 處理 boolean 類型的屬性（checked, disabled, selected 等）
- 需要獲取元素的當前狀態
- 處理表單元素的動態值
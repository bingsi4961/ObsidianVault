---
title: 教學 var $old = jQuery.noConflict(true);
tags: [jQuery, 技術文件]

---

**[:arrow_right: 在 javascript 中，()() 是什麼意思 :arrow_left:](/wEza_2WqRweIiLGc4I0CFw)**


```javascript
var $old = jQuery.noConflict(true);
```
`var $old` - 這是在宣告一個變量，變量名前面的 $ 符號是一種常見的命名慣例，表示這個變量會存儲 jQuery 對象。

`jQuery.noConflict(true)` - 這是核心部分，它做了兩件重要的事：

==1. 釋放 jQuery 對全局變量 $ 的控制權==
==2. 因為參數是 true，它也會釋放對 jQuery 這個全局變量的控制權==

這行代碼通常在什麼場景下使用呢？想象一下這個情境：
```javascript
// 假設您的網站原本是這樣使用 jQuery
$('#myButton').click(function() {
    // 處理點擊事件
});

// 現在您需要使用另一個也用 $ 符號的庫
// 使用 noConflict 來避免衝突
var $old = jQuery.noConflict(true);

// 之後您可以這樣使用 jQuery
$old('#myButton').click(function() {
    // 處理點擊事件
});

// 而其他庫可以安全地使用 $ 符號
```

為什麼這很重要？
- 在現代網頁開發中，我們經常需要使用多個 JavaScript 庫
- 許多庫都喜歡使用 $ 作為簡短標識符
- 如果不處理這些衝突，可能導致代碼無法正常工作

使用 noConflict 的最佳實踐：
```javascript
// 推薦的寫法，確保代碼的獨立性
(function($) {
    // 在這個作用域內，$ 保證是 jQuery
    $('#element').hide();
}($old));  // 將保存的 jQuery 傳入
```

我們再另外探討關於 jQuery

當我們執行 `var $old = jQuery.noConflict(true)` 時，這行代碼會釋放 jQuery 對全局變量 `jQuery` 的控制權，因為我們傳入了 `true` 參數。

所以在這之後：
```javascript
// 這行代碼會出錯
jQuery('#myButton').click(function() {
    // 處理點擊事件
});
```
這段代碼會報錯，因為 `jQuery` 這個全局變量已經不再指向 jQuery 庫了。

正確的使用方式應該是：
```javascript
// 正確的寫法：使用保存的 $old 變量
$old('#myButton').click(function() {
    // 處理點擊事件
});
```

為了更好地理解，讓我們看看完整的執行過程：
```javascript
// 1. 一開始我們可以使用 $ 和 jQuery
$('#element').hide();        // 有效
jQuery('#element').hide();   // 有效

// 2. 執行 noConflict
var $old = jQuery.noConflict(true);

// 3. 之後的狀態
$('#element').hide();        // 無效 - $ 已被釋放
jQuery('#element').hide();   // 無效 - jQuery 已被釋放
$old('#element').hide();     // 有效 - 這是正確的使用方式
```

如果您只想釋放 `$` 而保留 `jQuery`，您可以這樣做：
```javascript
// 只釋放 $ 符號的控制權
var $old = jQuery.noConflict();  // 注意：沒有傳入 true

// 之後
$('#element').hide();        // 無效
jQuery('#element').hide();   // 有效 - jQuery 還在
$old('#element').hide();     // 有效
```
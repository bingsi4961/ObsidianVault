---
title: '關於 $(function(){})() 的誤解'
tags: [jQuery, JavaScript]

---

# jQuery DOM Ready 事件處理的基本概念

**[:arrow_right: 在 javascript 中，()() 是什麼意思 :arrow_left:](/wEza_2WqRweIiLGc4I0CFw)**

當我們在 jQuery 中使用 `$(function(){})` 時，這實際上是 `$(document).ready(function(){})` 的簡寫形式。這種寫法的目的是確保我們的程式碼會在 DOM 完全建立後，再自動執行。

```javascript
// 完整寫法
$(document).ready(function(){
    // DOM 操作的程式碼
});

// 簡寫形式 - 效果完全相同
$(function(){
    // DOM 操作的程式碼
});
```

# 關於 $(function(){})() 的誤解

許多人可能會認為 `$(function(){})()` 的意思是「等待 DOM 載入完成後立即執行函式」，跟 $(function(){}) 是一樣的意思，但這個理解是不完全正確的。讓我們來看看為什麼：

```javascript
// 當我們寫 $(function(){})() 時，實際上發生了這些事：
let jQueryObject = $(function(){
    // 這是一個 ready handler
});
// 此時得到的是一個 jQuery 物件，不是一個可執行的函式

jQueryObject();  // 試圖將 jQuery 物件當作函式來執行
```
※ 另外 `(function(){})()` 是 javascript 原生語法，我自已是覺得應該不能跟 jQuery 混用。


# 正確的寫法

如果我們想要在 DOM 載入完成後立即執行某個函式，正確的寫法應該是：

```javascript
$(function(){
    // 這個外層函式會等待 DOM 載入
    
    (function(){
        // 這個內層函式會在 DOM 載入完成後立即執行
        console.log("DOM 已載入，這段程式碼立即執行");
    })();
});
```

# 實務上的重要性

這個概念在實務上很重要，因為：

1. DOM 操作必須等待 DOM 結構完全建立才能執行
2. 程式碼的執行順序會影響功能的正確性

# 使用情境的建議

在實際開發中，建議根據需求選擇適當的寫法：

```javascript
// 情境 1：一般的 DOM ready 事件處理
$(function(){
    // DOM 載入後執行的程式碼
});

// 情境 2：需要立即執行的函式（IIFE）
(function(){
    // 立即執行的程式碼
})();

// 情境 3：需要在 DOM ready 後立即執行的函式
$(function(){
    (function(){
        // 結合兩種需求的程式碼
    })();
});
```
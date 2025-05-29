---
title: JavaScript filter() vs jQuery filter()
tags: [JavaScript, jQuery]

---

讓我來說明 JavaScript 的 `filter` 和 jQuery 的 `filter` 的差異：

1. JavaScript 的 `filter`：
```javascript
// 用於陣列（Array）操作
const numbers = [1, 2, 3, 4, 5];

// 找出所有大於 2 的數字 (會建立一個新的陣列)
const filteredNumbers = numbers.filter(num => num > 2);
console.log(filteredNumbers); // 輸出: [3, 4, 5]

// 找出所有偶數
const evenNumbers = numbers.filter(num => num % 2 === 0);
console.log(evenNumbers); // 輸出: [2, 4]
```

2. jQuery 的 `filter`：
```javascript
// 用於 DOM 元素的篩選
// 假設 HTML 如下：
// <div class="items">
//   <p>項目 1</p>
//   <p class="special">項目 2</p>
//   <p>項目 3</p>
//   <p class="special">項目 4</p>
// </div>

// 找出所有具有 special 類別的段落
$("p").filter(".special");  // 選取 "項目 2" 和 "項目 4"

// 使用函式進行篩選
$("p").filter(function(index) {
    return index % 2 === 0;  // 選取偶數索引的段落
});
```

主要差異：
1. 使用對象不同：
   - JavaScript `filter`：用於處理陣列資料
   - jQuery `filter`：用於篩選 DOM 元素集合

2. 回傳值不同：
   - JavaScript `filter`：==回傳新的陣列==
   - jQuery `filter`：回傳 jQuery 物件

3. 使用場景：
   - JavaScript `filter`：資料處理、陣列操作
   - jQuery `filter`：DOM 元素篩選、頁面元素操作

讓我舉個實際例子來說明兩者的差異：

```javascript
// 情境：有一個錯誤訊息陣列和一些 DOM 元素

// 1. JavaScript filter 用法：處理陣列資料
const errorMessages = ["錯誤1", "錯誤2", "錯誤3"];
const resultMessage = "這是錯誤1的說明";

// 篩選出不在 resultMessage 中的錯誤訊息
const uniqueErrors = errorMessages.filter(msg => !resultMessage.includes(msg));
console.log(uniqueErrors); // 輸出: ["錯誤2", "錯誤3"]

// 2. jQuery filter 用法：處理 DOM 元素
// HTML: 
// <div class="error-messages">
//   <p data-error="錯誤1">錯誤1說明</p>
//   <p data-error="錯誤2">錯誤2說明</p>
//   <p data-error="錯誤3">錯誤3說明</p>
// </div>

// 篩選出特定的錯誤訊息元素
$(".error-messages p").filter(function() {
    const errorText = $(this).attr("data-error");
    return !resultMessage.includes(errorText);
}).addClass("highlight");
```
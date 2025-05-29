---
title: includes()、indexOf()、$.inArray()
tags: [JavaScript, jQuery]

---

JavaScript 的 `includes()` 方法主要用在兩個地方：

1. 用在字串（String）上：
```javascript
let text = "Hello, World";
console.log(text.includes("World")); // 輸出: true
console.log(text.includes("world")); // 輸出: false（區分大小寫）
```

2. 用在陣列（Array）上：
```javascript
let fruits = ["蘋果", "香蕉", "橘子"];
console.log(fruits.includes("香蕉")); // 輸出: true
console.log(fruits.includes("西瓜")); // 輸出: false
```

值得注意的是，`includes()` 是 ES6（ECMAScript 2015）引入的新方法。在較舊的瀏覽器中，我們會使用其他方法來達到相同效果：

對於字串，可以使用 `indexOf()`：
```javascript
let text = "Hello, World";
console.log(text.indexOf("World") !== -1); // 輸出: true
```

對於陣列，也可以使用 `indexOf()`：
```javascript
let fruits = ["蘋果", "香蕉", "橘子"];
console.log(fruits.indexOf("香蕉") !== -1); // 輸出: true
```

至於 jQuery，它本身並沒有直接提供 `includes()` 方法，但是我們可以使用 jQuery 的 `inArray()` 方法來達到類似效果：
```javascript
let fruits = ["蘋果", "香蕉", "橘子"];
console.log($.inArray("香蕉", fruits) !== -1); // 輸出: true
```

`includes()` 方法的優點是語法更直觀，代碼更容易理解。它直接返回 true 或 false，不需要像 `indexOf()` 那樣還要額外判斷是否等於 -1。

你有特定的使用場景需求嗎？我可以針對你的需求提供更具體的建議。
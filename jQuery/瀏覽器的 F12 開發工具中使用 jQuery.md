---
title: 瀏覽器的 F12 開發工具中使用 jQuery
tags: [jQuery]

---

## 網站已載入 jQuery 的情況
如果你正在瀏覽的網站已經載入了 jQuery，你可以直接在 Console 中使用：

```javascript
// 檢查是否有 jQuery
typeof $ !== 'undefined' 

// 或 
typeof jQuery !== 'undefined'
```

## 網站沒有 jQuery 的情況
如果網站沒有載入 jQuery，你可以動態載入：

```javascript
// 方法 1：動態載入 jQuery
var script = document.createElement('script');
script.src = 'https://code.jquery.com/jquery-3.7.1.min.js';
document.head.appendChild(script);

// 載入完成後就可以使用 $ 或 jQuery
```

```javascript
// 方法 2：使用 fetch 載入並執行
fetch('https://code.jquery.com/jquery-3.7.1.min.js')
  .then(response => response.text())
  .then(data => eval(data))
  .then(() => {
    console.log('jQuery 載入完成');
    // 現在可以使用 $ 了
  });
```
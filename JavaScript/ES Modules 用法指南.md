---
title: ES Modules 用法指南
tags: [JavaScript]

---

# ES Modules 用法指南

ES Modules (ESM) 是 JavaScript 的官方標準模組系統，讓你能夠將程式碼組織成可重複使用的獨立單元。我來解釋如何在一個 JavaScript 檔案中引用另一個檔案的功能。

## 基本匯出 (Export)

首先，你需要了解如何從一個檔案匯出功能。假設你有一個共用的檔案 `utils.js`：

```javascript
// utils.js

// 具名匯出 (Named exports)
export function add(a, b) {
  return a + b;
}

export function multiply(a, b) {
  return a * b;
}

// 你也可以先宣告，再一次性匯出
function subtract(a, b) {
  return a - b;
}

const PI = 3.14159;

export { subtract, PI };

// 預設匯出 (Default export) - 每個模組只能有一個
export default function sayHello(name) {
  return `你好，${name}！`;
}
```

## 基本引入 (Import)

現在，在另一個檔案中，你可以使用 `import` 語法來引用這些功能：

```javascript
// main.js

// 引入具名匯出
import { add, multiply, PI } from './utils.js';
// 注意路徑必須包含 .js 副檔名 (在瀏覽器環境中)
// 相對路徑以 ./ 開頭

console.log(add(5, 3));         // 輸出: 8
console.log(multiply(4, 2));    // 輸出: 8
console.log(PI);                // 輸出: 3.14159

// 引入預設匯出 (可以自行命名)
import sayHi from './utils.js';
console.log(sayHi('小明'));     // 輸出: 你好，小明！

// 同時引入預設匯出和具名匯出
import sayHello, { subtract } from './utils.js';
console.log(subtract(10, 4));   // 輸出: 6
```

## 進階技巧

### 重命名匯入/匯出

你可以使用 `as` 關鍵字來重命名：

```javascript
// 匯出時重命名
export { subtract as minus };

// 匯入時重命名
import { add as sum, PI as piValue } from './utils.js';
console.log(sum(1, 2));        // 輸出: 3
console.log(piValue);          // 輸出: 3.14159
```

### 匯入所有內容

你可以用 `* as` 語法一次匯入模組的所有具名匯出：

```javascript
import * as Utils from './utils.js';

console.log(Utils.add(5, 3));        // 輸出: 8
console.log(Utils.PI);               // 輸出: 3.14159
console.log(Utils.subtract(10, 4));  // 輸出: 6
```

### 重新匯出

你可以從一個模組匯入功能，然後再從你的模組匯出：

```javascript
// math.js
export { add, multiply } from './utils.js';
export { default as greeting } from './utils.js';
```

## 在瀏覽器中使用

在 HTML 檔案中，你需要指定 `type="module"` 來使用 ES Modules：

```html
<script type="module" src="main.js"></script>

<!-- 也可以直接寫在 HTML 中 -->
<script type="module">
  import { add } from './utils.js';
  console.log(add(1, 2));
</script>
```

## 注意事項

1. ES Modules 是靜態的，這表示 `import` 和 `export` 語句必須位於頂層作用域，不能在條件語句或函數內部使用。
2. ES Modules 始終在==嚴格模式 (`'use strict'`)== 下運行。
3. 在瀏覽器環境中，模組路徑必須是完整的相對或絕對路徑，包含副檔名（如 `./utils.js`）。
4. ES Modules 會被==延遲執行 (相當於加上 `defer` 屬性)==，所以不會阻塞 HTML 解析。
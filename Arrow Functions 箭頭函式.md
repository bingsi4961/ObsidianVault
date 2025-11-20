---
date: 2025-11-06 11:00
aliases:
tags:
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[]]

---

## 核心等效方法

### Array.prototype 方法（現代 JavaScript）

- **select → map**：轉換集合中的每個元素
    
    ```javascript
    const numbers = [1, 2, 3];
    const doubled = numbers.map(num => num * 2); // [2, 4, 6]
    const doubled = numbers.map(num => { return num * 2 });
    ```
    
- **where → filter**：篩選集合中符合條件的元素
    
    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    const evenNumbers = numbers.filter(num => num % 2 === 0); // [2, 4]
    const evenNumbers = numbers.filter(num => {return num % 2 ===0 });
    ```
    
- **any → some**：檢查是否至少有一個元素符合條件
    
    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    const hasEven = numbers.some(num => num % 2 === 0); // true
    ```
    
- **all → every**：檢查是否所有元素都符合條件
    
    ```javascript
    const numbers = [2, 4, 6, 8];
    const allEven = numbers.every(num => num % 2 === 0); // true
    ```
    
- **first → find**：返回第一個符合條件的元素
    
    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    const firstEven = numbers.find(num => num % 2 === 0); // 2
    ```
    
- **firstIndex → findIndex**：返回第一個符合條件的元素索引
    
    ```javascript
    const numbers = [1, 2, 3, 4];
    const index = numbers.findIndex(num => num > 2); // 2 (即數值3的索引)
    ```
    
- **contains → includes**：檢查集合是否包含特定元素
    
    ```javascript
    const numbers = [1, 2, 3];
    const hasTwo = numbers.includes(2); // true
    const hasTwo = numbers.some(n => [2,6].includes(n)); // true
    ```
    

## 其他有用的陣列方法

- **reduce**：將集合縮減為單一值（可用於實現許多其他 LINQ 操作）
    
    ```javascript
    const numbers = [1, 2, 3, 4];
    const sum = numbers.reduce((total, num) => total + num, 0); // 10
    ```
    
- **forEach**：對每個元素執行操作（類似於 LINQ 的 ForEach）
    
    ```javascript
    const numbers = [1, 2, 3];
    numbers.forEach(num => console.log(num));
    ```
    
- **sort**：排序集合（類似於 LINQ 的 OrderBy）
	
	- [[sort() 箭頭函式]]
	
    ```javascript
    const numbers = [3, 1, 4, 2];
    numbers.sort((a, b) => a - b); // [1, 2, 3, 4]
    ```
    
- **flat**：扁平化嵌套陣列（類似於 LINQ 的 SelectMany）
    
    ```javascript
    const nested = [[1, 2], [3, 4]];
    const flattened = nested.flat(); // [1, 2, 3, 4]
    ```
    
- **flatMap**：先映射每個元素，然後扁平化結果（結合 map 和 flat）
    
    ```javascript
    const sentences = ["Hello world", "How are you"];
    const words = sentences.flatMap(s => s.split(' ')); // ["Hello", "world", "How", "are", "you"]
    ```
    

## jQuery 方法

jQuery 也提供了一些類似的方法，特別是當處理 DOM 元素集合時：

- **$.map**：類似於 Array.map，但適用於 jQuery 對象
    
    ```javascript
    const numbers = [1, 2, 3];
    const doubled = $.map(numbers, num => num * 2); // [2, 4, 6]
    ```
    
- **$.grep**：類似於 Array.filter
    
    ```javascript
    const numbers = [1, 2, 3, 4, 5];
    const evenNumbers = $.grep(numbers, num => num % 2 === 0); // [2, 4]
    ```
    
- **$.each**：類似於 forEach
    
    ```javascript
    $.each([1, 2, 3], (index, value) => console.log(value));
    ```
    
- **$('selector').filter()**：篩選 DOM 元素
    
    ```javascript
    $('div').filter('.active'); // 選擇有 "active" 類的 div
    ```
    
- **$('selector').find()**：在子元素中查找
    
    ```javascript
    $('div').find('p'); // 找到 div 內的所有 p 元素
    ```
    

## Lodash 庫提供的額外功能

如果您需要更完整的 LINQ 等效功能，可以考慮使用 Lodash 庫，它提供更多的函數式編程功能：

```javascript
// 安裝: npm install lodash
const _ = require('lodash');

// 組合多個操作，類似於 LINQ 鏈式調用
const result = _([1, 2, 3, 4, 5])
  .filter(x => x % 2 === 0)    // 過濾偶數
  .map(x => x * 10)            // 每個元素乘以10
  .take(1)                     // 只取第一個元素
  .value();                    // [20]
```

Lodash 提供的其他實用方法：

- `_.groupBy`：按條件分組（類似於 LINQ 的 GroupBy）
- `_.orderBy`：多條件排序（類似於 LINQ 的 OrderBy/ThenBy）
- `_.uniqBy`：基於條件去重（類似於 LINQ 的 Distinct）
- `_.takeWhile`：獲取元素直到條件不再滿足（類似於 LINQ 的 TakeWhile）
- `_.zipObject`：將兩個數組合併為物件（類似於 LINQ 的 Zip）

## 總結

雖然 JavaScript 和 jQuery 沒有完全相同的 LINQ 語法，但透過原生的 Array 方法和可選的庫如 Lodash，您可以實現幾乎所有 LINQ 功能，並以類似的方式處理集合資料。現代 JavaScript（ES6+）的方法涵蓋了大部分常見的需求，而 Lodash 則可以補充更專業和複雜的操作。
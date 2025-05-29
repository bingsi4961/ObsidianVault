---
title: 數組 (Array) 與類數組 (Array-like Object)
tags: [JavaScript]

---

# 數組與類數組

在 JavaScript 中，數組（Array）是一種用於存儲多個值的特殊對象，而類數組（Array-like Object）則是看起來像數組但實際上不是真正數組的對象。讓我詳細解釋這兩個概念。

## 數組（Array）的特點

數組是 JavaScript 中的一種內建對象，具有以下特點：

1. 可以存儲多個不同類型的值
2. 使用數字索引（從0開始）來訪問元素
3. 有 `length` 屬性，表示數組中元素的數量
4. 繼承自 `Array.prototype`，因此有許多內建方法如 `push()`、`pop()`、`map()`、`filter()` 等

我們可以通過多種方式創建數組：

```javascript
// 使用數組字面量
const fruits = ['蘋果', '香蕉', '橙子'];

// 使用 Array 構造函數
const numbers = new Array(1, 2, 3, 4, 5);

// 使用 Array.of() 方法
const colors = Array.of('紅', '綠', '藍');

// 使用 Array.from() 將類數組轉換為真正的數組
const chars = Array.from('Hello');
```

## 類數組（Array-like Object）的特點

類數組對象是指：
1. 有 `length` 屬性
2. 有數字索引屬性（0, 1, 2...）
3. 但不是真正的數組，沒有數組的方法

JavaScript 中常見的類數組對象包括：

- DOM 節點列表（NodeList），如 `document.querySelectorAll('div')`
- 函數中的 `arguments` 對象
- 字符串（String 對象）

例如：

```javascript
function example() {
  console.log(arguments);
  console.log(arguments.length);  // 正常工作
  console.log(arguments[0]);      // 正常工作
  
  // 錯誤！arguments 不是真正的數組，沒有 map 方法
  // arguments.map(item => item * 2); 
}

example(1, 2, 3);
```

## 數組與類數組的主要區別

1. **原型鏈不同**：
   - 數組的原型是 `Array.prototype`
   - 類數組的原型通常是 `Object.prototype`

2. **方法可用性**：
   - 數組可以使用所有數組方法（`push`, `pop`, `map` 等）
   - 類數組不能直接使用這些方法

3. **instanceof 運算符**：
   - `arr instanceof Array` 對真正的數組返回 `true`
   - 對類數組返回 `false`

4. **Array.isArray()**：
   - `Array.isArray(arr)` 對真正的數組返回 `true`
   - 對類數組返回 `false`

## 將類數組轉換為真正的數組

我們可以使用以下方法將類數組轉換為真正的數組：

```javascript
// 使用 Array.from()（ES6+，推薦方式）
const arrayLike = {0: 'a', 1: 'b', 2: 'c', length: 3};
const realArray1 = Array.from(arrayLike);

// 使用展開運算符（ES6+）
const realArray2 = [...arrayLike];  // 注意：這只適用於可迭代的類數組

// 使用 Array.prototype.slice
const realArray3 = Array.prototype.slice.call(arrayLike);
```

將類數組轉換為真正的數組後，`我們就可以使用所有數組方法`，大大提高代碼的靈活性和可讀性。

## `length` 屬性的特殊含義

在 JavaScript 中，真正的數組有一個名為 `length` 的特殊屬性，它表示數組中元素的數量。當我們創建一個類數組對象時，`length` 屬性模擬了真正數組的這個行為。

```javascript
const arrayLike = { 
  0: 'apple',  // 數字索引屬性
  1: 'banana', // 數字索引屬性 
  2: 'cherry', // 數字索引屬性
  length: 3    // 特殊的長度屬性
};
```

## 為什麼 `length` 屬性特別重要

1. **定義元素範圍**：`length` 告訴像 `Array.from()` 這樣的方法應該處理多少個元素。如果沒有 `length` 屬性，`Array.from()` 就無法知道應該轉換多少個元素。

2. **必要條件**：要成為類數組對象，必須具有 `length` 屬性和以 0 為基礎的數字索引屬性。缺少 `length` 屬性的對象不會被視為類數組對象。

3. **迭代控制**：當方法迭代類數組對象時，它們會使用 `length` 屬性來決定迭代的次數，就像處理真正的數組一樣。

## 一個實驗來證明 `length` 的重要性

讓我們看看如果沒有 `length` 屬性會發生什麼：

```javascript
// 有 length 屬性的類數組對象
const withLength = { 0: 'apple', 1: 'banana', 2: 'cherry', length: 3 };
console.log(Array.from(withLength)); // ['apple', 'banana', 'cherry']

// 沒有 length 屬性的類似對象
const withoutLength = { 0: 'apple', 1: 'banana', 2: 'cherry' };
console.log(Array.from(withoutLength)); // [] (空數組!)
```

## `length` 屬性如何影響迭代

當 JavaScript 內部迭代類數組對象時，它會：
1. 讀取 `length` 屬性的值
2. 從索引 0 開始，一直迭代到 `length-1`
3. 獲取每個數字索引對應的值

這意味著 `length` 屬性實際上決定了有多少元素被認為是"存在的"，即使物理上對象中可能有更多或更少的屬性。

## `length` 值不匹配的情況

如果 `length` 值與實際數字索引屬性數量不匹配會發生什麼？

```javascript
// length 大於實際元素數量
const tooLong = { 0: 'apple', 1: 'banana', length: 5 };
console.log(Array.from(tooLong)); 
// ['apple', 'banana', undefined, undefined, undefined]

// length 小於實際元素數量
const tooShort = { 0: 'apple', 1: 'banana', 2: 'cherry', 3: 'date', length: 2 };
console.log(Array.from(tooShort)); 
// ['apple', 'banana'] (忽略了索引 2 和 3)
```

## 比較與普通屬性的不同

其他屬性（如 `color` 或 `type`）只是普通的鍵值對，不會影響對象的"類數組性質"：

```javascript
const arrayLikeWithExtras = { 
  0: 'apple',
  1: 'banana',
  2: 'cherry',
  color: 'red',  // 這個屬性被忽略  
  type: 'fruit',  // 這個屬性也被忽略
  length: 5  
};

console.log(Array.from(arrayLikeWithExtras)); 
// ['apple', 'banana', 'cherry', undefined, undefined] (只有數字索引屬性被處理)
```

總結來說，`length` 屬性是類數組對象的核心屬性，它定義了對象的"數組特性"，並控制了像 `Array.from()` 這樣的方法如何與對象交互。這使得它在功能上與其他普通屬性有本質的不同。
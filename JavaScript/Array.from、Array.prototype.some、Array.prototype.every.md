---
title: Array.from、Array.prototype.some、Array.prototype.every
tags: [JavaScript]

---

# Array.from 和 Array.prototype.some 詳解

[數組 (Array) 與類數組 (Array-like Object)]([//7rK4MoyeRD-5X-qIAX4hQQ?view](https://hackmd.io/7rK4MoyeRD-5X-qIAX4hQQ?view=))

## Array.from 方法

`Array.from()` 是一個能將類數組對象或可迭代對象轉換為真正的數組的靜態方法。

### 基本用法

```javascript
// 從字符串創建數組
const arr1 = Array.from('hello');
console.log(arr1); // ['h', 'e', 'l', 'l', 'o']

// 從 Set 創建數組
const set = new Set([1, 2, 3, 2, 1]);
const arr2 = Array.from(set);
console.log(arr2); // [1, 2, 3]

// 從類數組對象創建數組
const arrayLike = {
  0: 'apple',
  1: 'banana',
  2: 'cherry',
  length: 3
};
const arr3 = Array.from(arrayLike);
console.log(arr3); // ['apple', 'banana', 'cherry']
```

### 使用映射函數

`Array.from` 可以接受第二個參數 —— 一個映射函數，對每個元素進行處理：

```javascript
// 將數字數組的每個元素翻倍
const numbers = [1, 2, 3, 4];
const doubled = Array.from(numbers, x => x * 2);
console.log(doubled); // [2, 4, 6, 8]

// 將字符串轉換為字符數組並轉為大寫
const str = 'javascript';
const upperChars = Array.from(str, char => char.toUpperCase());
console.log(upperChars); // ['J', 'A', 'V', 'A', 'S', 'C', 'R', 'I', 'P', 'T']
```

### 生成序列數組

`Array.from` 常用於生成數字序列：

```javascript
// 創建 0 到 9 的數組
const sequence = Array.from({ length: 10 }, (_, i) => i);
console.log(sequence); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// 創建 1 到 10 的數組
const oneToTen = Array.from({ length: 10 }, (_, i) => i + 1);
console.log(oneToTen); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

## Array.prototype.some 方法

`Array.prototype.some()` 是一個測試數組中是否至少有一個元素通過指定測試函數的方法。

### 基本用法

```javascript
// 檢查數組是否包含偶數
const numbers = [1, 3, 5, 7, 8, 9];
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true，因為 8 是偶數

// 檢查數組是否包含大於 10 的數字
const smallNumbers = [1, 2, 3, 4, 5];
const hasLargeNumber = smallNumbers.some(num => num > 10);
console.log(hasLargeNumber); // false，沒有元素大於 10
```

### 實際應用場景

```javascript
// 檢查用戶列表中是否有管理員
const users = [
  { name: 'Alice', role: 'user' },
  { name: 'Bob', role: 'user' },
  { name: 'Charlie', role: 'admin' }
];
const hasAdmin = users.some(user => user.role === 'admin');
console.log(hasAdmin); // true

// 檢查產品列表中是否有庫存為 0 的商品
const products = [
  { name: 'Phone', stock: 10 },
  { name: 'Laptop', stock: 5 },
  { name: 'Tablet', stock: 0 }
];
const hasOutOfStock = products.some(product => product.stock === 0);
console.log(hasOutOfStock); // true
```

### 與 Array.prototype.every 的比較

`some()` 和 `every()` 是相對的方法：
- `some()`: 檢查是否「至少有一個」元素滿足條件（OR 邏輯）
- `every()`: 檢查是否「所有」元素都滿足條件（AND 邏輯）

```javascript
const scores = [75, 80, 95, 63, 88];

// 檢查是否至少有一個分數達到 90 分
const hasPassed90 = scores.some(score => score >= 90);
console.log(hasPassed90); // true，因為 95 >= 90

// 檢查是否所有分數都達到 60 分
const allPassed = scores.every(score => score >= 60);
console.log(allPassed); // true，所有分數都 >= 60
```

## 兩者結合使用的範例

```javascript
// 檢查分組數據中是否有符合特定條件的組
const groups = [
  [1, 3, 5],
  [2, 4, 6],
  [7, 9, 11]
];

// 檢查是否有一個組全部都是偶數
const hasAllEvenGroup = groups.some(group => 
  group.every(num => num % 2 === 0)
);
console.log(hasAllEvenGroup); // true，第二個組 [2, 4, 6] 全是偶數
```

## Array.from(非數組物件)

### 類數組對象的必要條件

一個對象要被視為類數組對象（array-like object），必須滿足兩個關鍵條件：

1. 擁有 `length` 屬性，表示元素的數量
2. 使用從0開始的連續整數索引來儲存元素

你提供的對象無法滿足這些條件：
```javascript
const arrayLike = { 
  name: 'Ethan',
  age: 42,
  addr: '中和區',
  phone: '0960949168'
};
```

這個對象使用的是描述性的屬性名（`name`、`age` 等），而不是數字索引（`0`、`1` 等），並且完全沒有 `length` 屬性。

### 嘗試使用 Array.from() 的結果

如果對這個對象使用 `Array.from()`，會發生什麼呢？

```javascript
const result = Array.from(arrayLike);
console.log(result); // []
```

結果會是一個空數組 `[]`。這是因為：

1. JavaScript 首先檢查對象是否有類似數組的結構
2. 發現它沒有 `length` 屬性，所以假定長度為 0
3. 然後嘗試查找從索引 0 開始的元素（找不到任何元素）
4. 最終返回一個空數組

### Array.from() 對迭代對象的處理

值得注意的是，`Array.from()` 不僅可以處理類數組對象，還可以處理任何可迭代對象（iterable）。但你的對象既不是類數組對象，也沒有實現迭代協議（沒有 `Symbol.iterator` 方法），所以不可迭代。

### 如何修改這個對象使其可用於 Array.from()

要讓這個對象可以使用 `Array.from()` 成功轉換，你有幾種選擇：

1. **轉換為類數組對象**：
```javascript
const arrayLike = { 
  0: 'Ethan',
  1: 42,
  2: '中和區',
  3: '0960949168',
  length: 4
};

const result = Array.from(arrayLike);
console.log(result); // ['Ethan', 42, '中和區', '0960949168']
```

2. **讓對象可迭代**：
```javascript
const iterableObject = { 
  name: 'Ethan',
  age: 42,
  addr: '中和區',
  phone: '0960949168',
  [Symbol.iterator]: function* () {
    yield this.name;
    yield this.age;
    yield this.addr;
    yield this.phone;
  }
};

const result = Array.from(iterableObject);
console.log(result); // ['Ethan', 42, '中和區', '0960949168']
```

### 對象屬性與數組元素的本質區別

這個例子很好地說明了 JavaScript 中對象屬性和數組元素的本質區別：

1. **數組元素**：順序性儲存，使用連續的數字索引訪問，有長度（`length`）概念
2. **對象屬性**：無序鍵值對，使用任意字符串或符號作為鍵，沒有內建的長度或順序概念

JavaScript 數組實際上是一種特殊的對象，它維護了一個特殊的 `length` 屬性並實現了迭代協議，這使得它們可以被迭代和索引訪問。

### 從普通對象值創建數組的正確方法

如果你想從普通對象的值創建數組，應該使用：

```javascript
const person = { 
  name: 'Ethan',
  age: 42,
  addr: '中和區',
  phone: '0960949168'
};

// 獲取所有值的數組
const valuesArray = Object.values(person);
console.log(valuesArray); // ['Ethan', 42, '中和區', '0960949168']

// 或者獲取所有鍵的數組
const keysArray = Object.keys(person);
console.log(keysArray); // ['name', 'age', 'addr', 'phone']

// 或者獲取鍵值對數組
const entriesArray = Object.entries(person);
console.log(entriesArray); 
// [['name', 'Ethan'], ['age', 42], ['addr', '中和區'], ['phone', '0960949168']]
```
---
date: 2025-06-05 18:32
aliases: 
tags:
  - JavaScript
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## sort() 方法基本概念

JavaScript 的 `Array.prototype.sort()` 方法用於對陣列元素進行排序。默認情況下，`sort()` 會將元素轉換為字串並按照 Unicode 編碼順序來排序，這對於數字排序常常不是我們想要的結果。

例如：

```javascript
[10, 2, 1, 21].sort(); // 結果為 [1, 10, 2, 21]
```

這不是正確的數值大小排序，因為它將數字轉換為字串後比較。

## 比較函數的作用

為了解決這個問題，`sort()` 方法可以接受一個比較函數作為參數，用來定義元素的排序方式。

```javascript
numbers.sort((a, b) => a - b);
```

在這個例子中：

- `(a, b) => a - b` 是一個箭頭函數（ES6 語法），等效於傳統的 `function(a, b) { return a - b; }`
- 這個函數接收兩個參數 `a` 和 `b`，它們代表陣列中被比較的兩個元素
- 函數返回 `a - b` 的結果，這個結果決定了 `a` 和 `b` 的排序順序

## 比較函數的回傳值含義

1. **如果返回值 < 0**：表示 `a` 應該排在 `b` 前面（升序）
2. **如果返回值 = 0**：表示 `a` 和 `b` 的順序不變
3. **如果返回值 > 0**：表示 `b` 應該排在 `a` 前面

所以在 `(a, b) => a - b` 中：

- 當 `a < b` 時，結果為負數，所以 `a` 排在 `b` 前面
- 當 `a = b` 時，結果為零，順序不變
- 當 `a > b` 時，結果為正數，所以 `b` 排在 `a` 前面

## 實例分析

以 `[3, 1, 4, 2]` 為例，讓我們看看排序過程：

1. 比較 3 和 1：`3 - 1 = 2` (正數)，所以 1 排在 3 前面 → `[1, 3, 4, 2]`
2. 比較 3 和 4：`3 - 4 = -1` (負數)，所以 3 排在 4 前面 → `[1, 3, 4, 2]`
3. 比較 4 和 2：`4 - 2 = 2` (正數)，所以 2 排在 4 前面 → `[1, 3, 2, 4]`
4. 比較 3 和 2：`3 - 2 = 1` (正數)，所以 2 排在 3 前面 → `[1, 2, 3, 4]`

最終結果：`[1, 2, 3, 4]`

## 降序排序

如果想要降序排序（從大到小），只需將比較函數改為：

```javascript
numbers.sort((a, b) => b - a);
```

這樣當 `b > a` 時，結果為正數，`a` 將排在 `b` 前面。

## 應用於其他類型

對於非數字或複雜物件，您需要根據實際情況調整比較函數：

```javascript
// 物件排序範例：依據 age 屬性升序排列
const people = [
  { name: "小明", age: 25 },
  { name: "小華", age: 18 },
  { name: "小英", age: 35 }
];

people.sort((a, b) => a.age - b.age);
// 結果：[{name: "小華", age: 18}, {name: "小明", age: 25}, {name: "小英", age: 35}]
```

## 注意事項

1. `sort()` 方法會直接修改原始陣列，這是一個「in-place」操作
2. 如果需要保留原始陣列，可以先複製一份：`[...numbers].sort((a, b) => a - b)`

這就是 `numbers.sort((a, b) => a - b)` 的詳細解析，它是在 JavaScript 中實現數字陣列按大小排序的標準方法。

## 直接比較字串

當您需要比較物件中的字串屬性（如 `name`）時，不能使用減法運算符，因為字串不適合用數學運算比較。我們可以直接使用比較運算符（ <, >, === ）來比較字串，字串比較是按照字典序進行的。

```javascript
const people = [
  { name: "張三", age: 25 },
  { name: "李四", age: 18 },
  { name: "王五", age: 35 }
];

// 按 name 字母順序（字典序）排序
people.sort((a, b) => {
  if (a.name < b.name) return -1;
  if (a.name > b.name) return 1;
  return 0;
});
```

## 使用 localeCompare 方法（推薦）

對於字串比較，特別是包含非英文字符的字串，使用 `String.prototype.localeCompare()` 方法是更好的選擇，因為它能夠正確處理不同語言的字符排序，包括中文字符。

```javascript
const people = [
  { name: "張三", age: 25 },
  { name: "李四", age: 18 },
  { name: "王五", age: 35 }
];

// 使用 localeCompare 進行排序
people.sort((a, b) => a.name.localeCompare(b.name, 'zh-TW'));
```

這裡的 `'zh-TW'` 是語言區域設定，用於指定台灣繁體中文的排序規則。`localeCompare` 方法會返回：

- 負數：如果參考字串（a.name）在比較字串（b.name）之前
- 0：如果兩個字串相等
- 正數：如果參考字串在比較字串之後

## 不區分大小寫的比較

如果您需要不區分大小寫地比較字串，可以將字串轉換為相同的大小寫後再比較：

```javascript
// 不區分大小寫排序
people.sort((a, b) => a.name.toLowerCase().localeCompare(b.name.toLowerCase(), 'zh-TW'));
```

## 處理中文拼音排序

對於中文字符，如果希望按照拼音順序排序而非筆畫順序，您可以使用 `localeCompare` 的詳細選項：

```javascript
people.sort((a, b) => a.name.localeCompare(b.name, 'zh-TW', { 
  sensitivity: 'accent',  // 區分重音但不區分大小寫
  caseFirst: 'false'      // 不特別考慮大小寫順序
}));
```

## 多屬性排序

如果需要先按 `name` 排序，相同 `name` 的再按 `age` 排序：

```javascript
people.sort((a, b) => {
  // 先比較 name
  const nameComparison = a.name.localeCompare(b.name, 'zh-TW');
  
  // 如果 name 相同，再比較 age
  if (nameComparison === 0) {
    return a.age - b.age;
  }
  
  return nameComparison;
});
```

## 使用排序函數的實際例子

```javascript
const employees = [
  { name: "陳小明", department: "行銷", age: 28 },
  { name: "林大偉", department: "研發", age: 32 },
  { name: "陳小明", department: "行銷", age: 24 },  // 同名不同年齡
  { name: "王美麗", department: "人資", age: 35 },
  { name: "李志成", department: "研發", age: 27 }
];

// 按照部門、姓名、年齡排序
employees.sort((a, b) => {
  // 先按部門排序
  const deptComparison = a.department.localeCompare(b.department, 'zh-TW');
  if (deptComparison !== 0) return deptComparison;
  
  // 部門相同則按姓名排序
  const nameComparison = a.name.localeCompare(b.name, 'zh-TW');
  if (nameComparison !== 0) return nameComparison;
  
  // 姓名相同則按年齡排序
  return a.age - b.age;
});

console.log(employees);
// 結果將會是：
// [
//   { name: "王美麗", department: "人資", age: 35 },
//   { name: "陳小明", department: "行銷", age: 24 },
//   { name: "陳小明", department: "行銷", age: 28 },
//   { name: "李志成", department: "研發", age: 27 },
//   { name: "林大偉", department: "研發", age: 32 }
// ]
```

## 注意事項

1. 當處理中文或其他非英文字符時，始終建議使用 `localeCompare` 以獲得正確的排序結果
2. 在處理大量數據時，字串比較可能比數值比較更耗性能
3. 如果需要頻繁排序，考慮將排序結果緩存或預先計算
4. 確保比較的字串屬性存在，避免空值或 undefined 引起錯誤

在現代 JavaScript 中，`localeCompare` 是處理字串排序的最佳實踐，特別是處理多語言內容時。它能確保您的應用程式在不同語言環境下都能提供一致且符合當地習慣的排序結果。
---
date: 2025-11-27 15:08
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
# 連結筆記
#### 📑 [[]]

---
## 核心概念

`reduce()` 是 JavaScript 陣列方法，作用是將陣列所有元素透過計算邏輯「歸納」成單一個值。

**C# 對照**：等同於 LINQ 的 `Aggregate()` 方法。

```javascript
// JavaScript
const sum = numbers.reduce((total, num) => total + num, 0);

// C# 對照
var sum = numbers.Aggregate(0, (total, num) => total + num);
```

---
## 基本語法與參數

```javascript
array.reduce((accumulator, currentValue) => { ... }, initialValue);
```

|參數|說明|
|---|---|
|`accumulator` (累加器)|記憶上一次迴圈計算完的結果，類似 foreach 外面宣告的 `int sum = 0`|
|`currentValue` (當前值)|目前正輪到的陣列元素|
|`initialValue` (初始值)|累加器在第一次執行時的預設值|

**執行流程範例**：`[1, 2, 3, 4].reduce((total, num) => total + num, 0)`

|迴圈次數|total|num|計算|新 total|
|---|---|---|---|---|
|第 1 次|0 (初始值)|1|0 + 1|1|
|第 2 次|1|2|1 + 2|3|
|第 3 次|3|3|3 + 3|6|
|第 4 次|6|4|6 + 4|**10**|

---
## 初始值設定口訣

|目的|初始值|
|---|---|
|加總/計數 (Sum/Count)|`0`|
|過濾並重組 (Filter + Map)|`[]`|
|分組 (Group By)|`{}`|

⚠️ **重要**：處理物件陣列時務必給初始值，否則會導致錯誤。

---
## 實務應用：計算總金額

```javascript
const cart = [
  { product: "筆電", price: 30000 },
  { product: "滑鼠", price: 1000 },
  { product: "鍵盤", price: 2000 }
];

const totalPrice = cart.reduce((total, item) => {
    return total + item.price;
}, 0);

console.log(totalPrice); // 33000
```

---
## 進階應用：資料分組 (Group By)

使用 `reduce` 實作類似 SQL 的 Group By 功能，產出結構等同於 C# 的 `Dictionary<string, List<T>>`。

```javascript
const employees = [
  { id: 1, name: "Jason", dept: "RD" },
  { id: 2, name: "May",   dept: "Sales" },
  { id: 3, name: "Alex",  dept: "RD" },
  { id: 4, name: "Tim",   dept: "HR" },
  { id: 5, name: "Ben",   dept: "Sales" }
];

const groupedData = employees.reduce((acc, curr) => {
  const key = curr.dept;
  
  if (!acc[key]) {
    acc[key] = [];
  }
  
  acc[key].push(curr);
  return acc;
}, {}); // 初始值是空物件
```

**簡潔寫法**：

```javascript
const grouped = employees.reduce((acc, curr) => {
  (acc[curr.dept] = acc[curr.dept] || []).push(curr);
  return acc;
}, {});
```

**輸出結果**：

```json
{
  "RD": [
    { "id": 1, "name": "Jason", "dept": "RD" },
    { "id": 3, "name": "Alex",  "dept": "RD" }
  ],
  "Sales": [
    { "id": 2, "name": "May",   "dept": "Sales" },
    { "id": 5, "name": "Ben",   "dept": "Sales" }
  ],
  "HR": [
    { "id": 4, "name": "Tim",   "dept": "HR" }
  ]
}
```

**C# 對照**：

```csharp
var groupedData = employees
    .GroupBy(x => x.dept)
    .ToDictionary(g => g.Key, g => g.ToList());
```

---
## 分組結果的讀取方式

```javascript
// 直接取得特定部門
var rdTeam = groupedData["RD"]; 
// 或 groupedData.RD

// 遍歷所有分組
var depts = Object.keys(groupedData); // ["RD", "Sales", "HR"]

depts.forEach(function(deptName) {
    var employees = groupedData[deptName];
    console.log("=== " + deptName + " ===");
    employees.forEach(function(emp) {
        console.log(emp.name);
    });
});
```

---
## 點記法 vs 括號記法

JavaScript 物件同時也是字典，以下兩種寫法等價：

```javascript
groupedData["RD"]  // 括號記法 (C# Dictionary 風格)
groupedData.RD     // 點記法 (C# Class 風格)
```

**只能用括號記法的情況**：

|情況|範例|
|---|---|
|Key 包含空白或特殊符號|`groupedData["RD Team"]`、`groupedData["R&D"]`|
|Key 是數字開頭|`groupedData["2025"]`|
|Key 來自變數|`var dept = "RD"; groupedData[dept]`|

⚠️ **常見錯誤**：

```javascript
var targetDept = "RD";
groupedData.targetDept  // ❌ undefined (找的是字面屬性 "targetDept")
groupedData[targetDept] // ✅ 正確 (使用變數值 "RD")
```

---
## 關於 `||=` 運算子

JavaScript ES2021 引入了 `||=`（邏輯 OR 指派），類似 C# 的 `??=`：

```javascript
(acc[curr.dept] ||= []).push(curr);
```

**⚠️ 不建議在 Portal/GTS 系統使用**：

- `||=` 是 ES2021 語法，不支援 IE 及 2020 年前的舊瀏覽器
- 除非專案有 Babel/TypeScript 編譯，否則會導致程式崩潰

**安全替代寫法**：

```javascript
// 選項 A：簡寫 (推薦，相容性極好)
(acc[curr.dept] = acc[curr.dept] || []).push(curr);

// 選項 B：傳統 IF (最穩，最好維護)
if (!acc[curr.dept]) {
    acc[curr.dept] = [];
}
acc[curr.dept].push(curr);
```

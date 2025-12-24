---
date: 2025-11-27 17:39
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
## 一、基本概念與陷阱

### 1.1 預設行為的問題

JavaScript 的 `Array.prototype.sort()` 預設會將元素**轉換為字串**，按照 **Unicode 編碼**順序排序。

```javascript
// ⚠️ 錯誤示範：數字排序
[10, 2, 1, 21].sort();  
// 結果：[1, 10, 2, 21]  ← 因為 "1" < "10" < "2" < "21"

// ⚠️ 錯誤示範：不傳比較函式
const nums = [10, 2, 5];
nums.sort();  
// 結果：[10, 2, 5]  ← 字串排序
```

### 1.2 解決方案：比較函式

```javascript
// ✓ 正確：數值排序
[10, 2, 1, 21].sort((a, b) => a - b);  
// 結果：[1, 2, 10, 21]
```

### 1.3 重要特性

| 特性              | 說明                             |
| --------------- | ------------------------------ |
| **In-place 操作** | ==直接修改原始陣列==                   |
| **保留原陣列**       | 使用 ==[...array].sort()== 複製後排序 |
| **穩定排序**        | 相等元素保持原有順序（現代瀏覽器）              |


---
## 二、比較函式核心原理（黃金規則）

### 2.1 回傳值的意義

```javascript
array.sort((a, b) => { 回傳值 })
```

🚨 回傳負數 → a 排在前
🚨 回傳正數 → b 排在前
🚨 回傳零   → 不變

- 參數名稱：不管參數叫 a, b 還是 x, y，定義永遠是「第一個參數 vs 第二個參數」。
- 函式回傳負數：意思就是「第一個參數 贏了，它要排前面」。
- 函式回傳正數：意思就是「第二個參數 贏了，它要排前面」。

### 2.2 a 及 b 是指哪一個？

在 `[3, 1, 4, 2]` 中，**a 和 b 不是固定位置**，而是每次比較時，系統隨機抽出的兩個元素。

#### 簡單理解

```
a = 當前比較的第一個值
b = 當前比較的第二個值
```

#### 重點提醒

- **不要管它們原本在陣列的哪個位置**
- 每次比較時，a 和 b 都可能是不同的元素
- 只需要專注在「如何比較這兩個值」即可

#### 實際範例

```javascript
[3, 1, 4, 2].sort((a, b) => a - b);
```

某次比較可能是：

- a = 3, b = 1
- a = 4, b = 2
- a = 1, b = 4
- ...（依演算法決定）

---
## 三、數值排序實戰

### 3.1 基本語法

```javascript
// 升冪（由小到大）
numbers.sort((a, b) => a - b);

// 降冪（由大到小）
numbers.sort((a, b) => b - a);
```

🚨  **關鍵理解：** 系統不知道「升降冪」概念，只看回傳正負號決定位置！

### 3.2 執行過程詳解

**範例：`[3, 1, 4, 2]` 升冪排序**

```javascript
const numbers = [3, 1, 4, 2];
numbers.sort((a, b) => a - b);
```

| 回合  | 比較     | 計算     | 結果  | 判斷   | 目前陣列             |
| --- | ------ | ------ | --- | ---- | ---------------- |
| 1   | 3 vs 1 | 3-1=2  | 正數  | 1排在前 | `[1, 3, 4, 2]`   |
| 2   | 3 vs 4 | 3-4=-1 | 負數  | 3排在前 | `[1, 3, 4, 2]`   |
| 3   | 4 vs 2 | 4-2=2  | 正數  | 2排在前 | `[1, 3, 2, 4]`   |
| 4   | 3 vs 2 | 3-2=1  | 正數  | 2排在前 | `[1, 2, 3, 4]`   |
| 5   | 1 vs 2 | 1-2=-1 | 負數  | 1排在前 | `[1, 2, 3, 4]` ✓ |
|     |        |        |     |      |                  |


---
## 四、字串排序

當您需要比較物件中的字串屬性（如 `name`）時，不能使用減法運算符，因為字串不適合用數學運算比較。
我們可以使用比較運算符（ <, >, === ）來比較字串，字串比較是按照字典序進行的，==**而字典序的比較基礎是 Unicode 編碼值**。

### 4.1 基礎比較（不推薦）

```javascript
const people = [
  { name: "五王", age: 25 },
  { name: "三張", age: 18 },
  { name: "四李", age: 35 }
];

// 基礎寫法（按 Unicode 編碼）
people.sort((a, b) => {
  if (a.name < b.name) return -1;
  if (a.name > b.name) return 1;
  return 0;
});

// 結果：「三張、五王、四李」
// 因為 四(22235) > 五(20116) 的 Unicode 編碼
```

### 4.2 localeCompare 方法（✓ 推薦）

對於字串比較，特別是包含非英文字符的字串，使用 `String.prototype.localeCompare()` 方法是更好的選擇
因為它能夠正確處理不同語言的字符排序，包括中文字符。

#### 基本用法

```javascript
const people = [
  { name: "張三", age: 25 },
  { name: "李四", age: 18 },
  { name: "王五", age: 35 }
];

// 使用 localeCompare（適用中文）
people.sort((a, b) => a.name.localeCompare(b.name, 'zh-TW'));
```

這裡的 `'zh-TW'` 是語言區域設定，用於指定台灣繁體中文的排序規則。

`localeCompare` 方法會返回：
- 負數：如果參考字串（a.name）在比較字串（b.name）之前
- 0：如果兩個字串相等
- 正數：如果參考字串在比較字串之後

#### 進階選項

```javascript
// 不區分大小寫
people.sort((a, b) => 
  a.name.toLowerCase().localeCompare(b.name.toLowerCase(), 'zh-TW')
);

// 詳細設定（拼音排序）
people.sort((a, b) => 
  a.name.localeCompare(b.name, 'zh-TW', { 
    sensitivity: 'accent',  // 區分重音但不區分大小寫
    caseFirst: 'false'      // 不特別考慮大小寫順序
  })
);
```


---
## 五、物件陣列排序

### 5.1 單一屬性排序

```javascript
const employees = [
  { EmpID: 1003, Name: "張三" },
  { EmpID: 1001, Name: "李四" },
  { EmpID: 1002, Name: "王五" }
];

// 數值屬性：依工號升冪
employees.sort((emp1, emp2) => emp1.EmpID - emp2.EmpID);

// 字串屬性：依姓名排序
employees.sort((emp1, emp2) => 
  emp1.Name.localeCompare(emp2.Name, 'zh-TW')
);
```

### 5.2 多重條件排序

```javascript
const employees = [
  { name: "陳小明", department: "行銷", age: 28 },
  { name: "林大偉", department: "研發", age: 32 },
  { name: "陳小明", department: "行銷", age: 24 },  // 同名不同年齡
  { name: "王美麗", department: "人資", age: 35 },
  { name: "李志成", department: "研發", age: 27 }
];

// 排序優先順序：部門 > 姓名 > 年齡
employees.sort((a, b) => {
  // 第一層：部門
  const deptComparison = a.department.localeCompare(b.department, 'zh-TW');
  if (deptComparison !== 0) return deptComparison;
  
  // 第二層：姓名
  const nameComparison = a.name.localeCompare(b.name, 'zh-TW');
  if (nameComparison !== 0) return nameComparison;
  
  // 第三層：年齡
  return a.age - b.age;
});

// 結果：
// [
//   { name: "王美麗", department: "人資", age: 35 },
//   { name: "陳小明", department: "行銷", age: 24 },
//   { name: "陳小明", department: "行銷", age: 28 },
//   { name: "李志成", department: "研發", age: 27 },
//   { name: "林大偉", department: "研發", age: 32 }
// ]
```

### 5.3 簡化寫法（連鎖比較）

```javascript
// 使用 || 運算符簡化多重條件
people.sort((a, b) => 
  a.name.localeCompare(b.name, 'zh-TW') || a.age - b.age
);
// 意義：如果 name 相同（回傳 0），就執行 age 比較
```


---
## 六、參數命名與可讀性

### 6.1 參數可以自訂名稱

```javascript
// 標準寫法
numbers.sort((a, b) => a - b);

// 自訂名稱（完全可行）
numbers.sort((num1, num2) => num1 - num2);
numbers.sort((x, y) => x - y);
```

### 6.2 實務建議

|情境|推薦命名|範例|
|---|---|---|
|**數字陣列**|`a, b`|`(a, b) => a - b`|
|**物件陣列**|語意化名稱|`(emp1, emp2) => emp1.Seniority - emp2.Seniority`|
|**複雜邏輯**|完整命名|`(employee1, employee2) => ...`|

```javascript
// ✓ 好的可讀性
employees.sort((emp1, emp2) => 
  emp1.Seniority - emp2.Seniority
);

// ✗ 可讀性較差
employees.sort((a, b) => a.Seniority - b.Seniority);
```


---
## 七、與 C# 的對照

### 7.1 語法對照表

| 功能            | JavaScript                                        | C#                                                                                 |
| ------------- | ------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **升冪**        | `arr.sort((a, b) => a - b)`                       | `list.Sort((a, b) => a.CompareTo(b))`<br>`list.OrderBy(x => x).ToList()`           |
| **降冪**        | `arr.sort((a, b) => b - a)`                       | `list.Sort((a, b) => b.CompareTo(a))`<br>`list.OrderByDescending(x => x).ToList()` |
| **字串排序**      | `arr.sort((a, b) => a.localeCompare(b, 'zh-TW'))` | `list.OrderBy(x => x, StringComparer.CurrentCulture)`                              |
| **==修改原陣列==** | ✓ 是（In-place）                                     | `Sort()` 是<br>`OrderBy()` 否                                                        |

### 7.2 概念對應

```csharp
// C# IComparer 介面
public int Compare(int x, int y) {
    return x.CompareTo(y);  // 等同於 JS 的 a - b
}
```

```javascript
// JavaScript 等效寫法
(a, b) => a - b
```

---

## 八、實務應用範例

### 8.1 Portal 系統：訂單排序

```javascript
// 假設這是從 API 取得的訂單資料
const orders = [
  { OrderID: 1003, OrderDate: "2024-03-15", TotalAmount: 15000 },
  { OrderID: 1001, OrderDate: "2024-03-10", TotalAmount: 8000 },
  { OrderID: 1002, OrderDate: "2024-03-12", TotalAmount: 12000 }
];

// 需求 1：依訂單日期（新到舊）
orders.sort((a, b) => 
  new Date(b.OrderDate) - new Date(a.OrderDate)
);

// 需求 2：依金額（高到低）
orders.sort((a, b) => b.TotalAmount - a.TotalAmount);

// 需求 3：複合排序（日期 > 金額）
orders.sort((a, b) => 
  new Date(b.OrderDate) - new Date(a.OrderDate) || 
  b.TotalAmount - a.TotalAmount
);
```

### 8.2 GTS 系統：零件庫存排序

```javascript
const parts = [
  { PartNo: "MB-001", Stock: 50, LastUpdate: "2024-03-01" },
  { PartNo: "CPU-002", Stock: 0, LastUpdate: "2024-03-05" },
  { PartNo: "RAM-003", Stock: 120, LastUpdate: "2024-03-03" }
];

// 需求：缺貨優先顯示，再依更新時間
parts.sort((a, b) => {
  // 第一層：缺貨（0）排在前面
  if (a.Stock === 0 && b.Stock !== 0) return -1;
  if (a.Stock !== 0 && b.Stock === 0) return 1;
  
  // 第二層：依最後更新時間（新到舊）
  return new Date(b.LastUpdate) - new Date(a.LastUpdate);
});
```

---

## 九、注意事項與最佳實踐

### 9.1 效能考量

```javascript
// 原始資料 
const people = [ 
	{ name: "張三", date: "2024-03-15" }, 
	{ name: "李四", date: "2024-01-20" }, 
	{ name: "王五", date: "2024-02-10" } 
];

// ✗ 效能不佳：每次比較都創建 Date 物件
people.sort((a, b) => 
  new Date(a.date) - new Date(b.date)
);

// ✓ 效能較好：預先轉換
people.forEach(p => p.dateObj = new Date(p.date));
people.sort((a, b) => a.dateObj - b.dateObj);
```

### 9.2 安全性檢查

```javascript
// ✓ 防止 null/undefined 錯誤
employees.sort((a, b) => {
  // 處理空值
  if (!a.name) return 1;  // a 排後面
  if (!b.name) return -1; // b 排後面
  
  return a.name.localeCompare(b.name, 'zh-TW');
});
```

### 9.3 保留原陣列

```javascript
// 方法 1：展開運算符
const sortedArray = [...originalArray].sort((a, b) => a - b);

// 方法 2：slice()
const sortedArray = originalArray.slice().sort((a, b) => a - b);

// 方法 3：Array.from()
const sortedArray = Array.from(originalArray).sort((a, b) => a - b);
```

---

## 十、常見錯誤與解決

### 10.1 忘記回傳值

```javascript
// ✗ 錯誤：沒有 return
numbers.sort((a, b) => { a - b });  // undefined

// ✓ 正確
numbers.sort((a, b) => a - b);
numbers.sort((a, b) => { return a - b; });
```

### 10.2 比較字串用減法

```javascript
// ✗ 錯誤：字串不能相減
people.sort((a, b) => a.name - b.name);  // NaN

// ✓ 正確
people.sort((a, b) => a.name.localeCompare(b.name, 'zh-TW'));
```

### 10.3 忽略 In-place 特性

```javascript
// ✗ 錯誤：以為 sort() 不改變原陣列
const original = [3, 1, 2];
const sorted = original.sort((a, b) => a - b);
console.log(original);  // [1, 2, 3] ← 原陣列被改變了！

// ✓ 正確：需要保留原陣列時
const sorted = [...original].sort((a, b) => a - b);
```

---

## 十一、總結重點

### 核心原則

1. **永遠傳入比較函式**（數值排序，否則會變字串排序）
2. **負數 a 在前，正數 b 在前**（鐵律）
3. **a - b 升冪，b - a 降冪**（數學魔術）
4. **字串用 localeCompare**（中文友善）
5. **In-place 操作**（會改變原陣列）

### 快速查詢表

```javascript
// 數值排序
arr.sort((a, b) => a - b)        // 升冪 ↑
arr.sort((a, b) => b - a)        // 降冪 ↓

// 字串排序（中文）
arr.sort((a, b) => a.localeCompare(b, 'zh-TW'))

// 物件排序
arr.sort((a, b) => a.prop - b.prop)
arr.sort((a, b) => a.prop.localeCompare(b.prop, 'zh-TW'))

// 保留原陣列
[...arr].sort((a, b) => a - b)
```

### 實務建議

| 情境        | 推薦做法                        |
| --------- | --------------------------- |
| **數值排序**  | `(a, b) => a - b`           |
| **中文排序**  | `localeCompare(b, 'zh-TW')` |
| **日期排序**  | `new Date(a) - new Date(b)` |
| **多重條件**  | 用 `                         |
| **保留原陣列** | 先複製 `[...arr]`              |
| **大量資料**  | 考慮預先轉換、快取                   |

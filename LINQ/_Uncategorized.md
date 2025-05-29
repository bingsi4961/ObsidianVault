---
title: _Uncategorized
tags: [LINQ]

---

```csharp=
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// Where
// 用途：過濾集合，返回符合條件的所有元素
// 返回：IEnumerable<T>、沒有符合的資料，會返回空集合，不會拋出異常。
// 延遲執行 (lazy evaluation)
var evenNumbers = numbers.Where(n => n % 2 == 0);
// 結果: { 2, 4 }

// Any
// 用途：檢查集合中是否存在符合條件的元素
// 返回：bool
// 立即執行
bool hasEvenNumber = numbers.Any(n => n % 2 == 0);
// 結果: true

// Find
// 用途：返回集合中【第一個】符合條件的元素
// 返回：T 或 default(T)
// 立即執行
int firstEvenNumber = numbers.Find(n => n % 2 == 0);
// 結果: 2

// Max
// 用途：取得 EmployeeID 欄位的最大值
// 返回：int
// 立即執行
// 如果 Employees 資料表是空的，會拋出 InvalidOperationException 
int maxEmpID = db.Employees.Max(e => e.EmployeeID);

// 為了避免 InvalidOperationException ，有幾種處理方式：
int maxEmpID = db.Employees
    .Select(e => e.EmployeeID)
    .DefaultIfEmpty(0)  // 如果序列為空，返回0
    .Max();

int maxEmpID = db.Employees.Any() ? db.Employees.Max(e => e.EmployeeID) : 0;


// ====================================================================

List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// FirstOrDefault: 返回第一個符合條件的元素，如果沒有則返回默認值
// 與 Find 的區別：可用於任何 IEnumerable<T>，而不僅僅是 List<T>。
int firstEven = numbers.FirstOrDefault(n => n % 2 == 0);
// 結果: 2

// SingleOrDefault: 返回唯一符合條件的元素，如果沒有或有多個則返回默認值
int singleFive = numbers.SingleOrDefault(n => n == 5);
// 結果: 5

// All: 檢查是否所有元素都符合條件
bool allPositive = numbers.All(n => n > 0);
// 結果: true

// Count: 計算符合條件的元素數量
// 可以用來替代 Where().Count()，效率更高。
int evenCount = numbers.Count(n => n % 2 == 0);
// 結果: 5

// Skip 和 Take: 用於分頁
var page = numbers.Skip(2).Take(3);
// 結果: { 3, 4, 5 }

// Select: 投影每個元素到新的形式
var squares = numbers.Select(n => n * n);
// 結果: { 1, 4, 9, 16, 25, 36, 49, 64, 81, 100 }

// Distinct: 去除重複元素
List<int> withDuplicates = new List<int> { 1, 2, 2, 3, 3, 4, 5 };
var unique = withDuplicates.Distinct();
// 結果: { 1, 2, 3, 4, 5 }
```

分別以 List<int>、context.Employees 為 source，請教學 Where 指令的使用方式
分別以 List<int>、context.Employees 為 source，請教學 Where 指令的延遲執行
分別以 List<int>、context.Employees 為 source，請教學 Where 指令的會發生的 Exception
    
    Select、Where、Distinct、First、FirstOrDefault、Single、SingleOrDefault、Find、Count、Any、All、Max、Min、Sum、Average、Take、Skip
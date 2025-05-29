---
title: _Where
tags: [LINQ]

---

# 延遲執行

1. 使用 List<int> 的延遲執行示範：

```csharp
// 建立資料來源
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// 建立查詢但尚未執行
IEnumerable<int> evenQuery = numbers.Where(n => {
    Console.WriteLine($"檢查數字: {n}");
    return n % 2 == 0;
});

Console.WriteLine("查詢已定義，但尚未執行");

// 修改原始集合
numbers.Add(6);

// 實際執行查詢（這時才會執行 Where 條件）
Console.WriteLine("開始執行查詢:");
foreach (int num in evenQuery)
{
    Console.WriteLine($"找到偶數: {num}");
}

// 再次執行查詢（會重新執行整個查詢）
Console.WriteLine("\n再次執行相同查詢:");
foreach (int num in evenQuery)
{
    Console.WriteLine($"找到偶數: {num}");
}

/* 輸出結果：
查詢已定義，但尚未執行
開始執行查詢:
檢查數字: 1
檢查數字: 2
找到偶數: 2
檢查數字: 3
檢查數字: 4
找到偶數: 4
檢查數字: 5
檢查數字: 6
找到偶數: 6

再次執行相同查詢:
檢查數字: 1
檢查數字: 2
找到偶數: 2
檢查數字: 3
檢查數字: 4
找到偶數: 4
檢查數字: 5
檢查數字: 6
找到偶數: 6
*/
```

2. 使用 Entity Framework context.Employees 的延遲執行示範：

```csharp
// 準備測試用的 DbContext 和資料
public class EmployeeContext : DbContext
{
    public DbSet<Employee> Employees { get; set; }
}

// 建立查詢但尚未執行
IQueryable<Employee> query = context.Employees.Where(e => {
    Console.WriteLine($"正在檢查員工資料");
    return e.Department == "IT";
});

Console.WriteLine("=== 查詢已定義，尚未存取資料庫 ===");

// 顯示產生的 SQL
Console.WriteLine("\n=== 產生的初始 SQL ===");
Console.WriteLine(query.ToQueryString());

// 添加額外條件
query = query.Where(e => e.Age > 30);
Console.WriteLine("\n=== 修改後的 SQL ===");
Console.WriteLine(query.ToQueryString());

// 實際執行查詢
Console.WriteLine("\n=== 開始執行資料庫查詢 ===");
List<Employee> results = query.ToList();  // 這裡才實際查詢資料庫

foreach (Employee emp in results)
{
    Console.WriteLine($"找到符合條件的員工: {emp.Name}, 年齡: {emp.Age}, 部門: {emp.Department}");
}

/* 輸出結果：
=== 查詢已定義，尚未存取資料庫 ===

=== 產生的初始 SQL ===
SELECT [e].[Id], [e].[Name], [e].[Age], [e].[Department], [e].[Salary]
FROM [Employees] AS [e]
WHERE [e].[Department] = N'IT'

=== 修改後的 SQL ===
SELECT [e].[Id], [e].[Name], [e].[Age], [e].[Department], [e].[Salary]
FROM [Employees] AS [e]
WHERE ([e].[Department] = N'IT') AND ([e].[Age] > 30)

=== 開始執行資料庫查詢 ===
找到符合條件的員工: John Smith, 年齡: 35, 部門: IT
找到符合條件的員工: Mary Johnson, 年齡: 42, 部門: IT
找到符合條件的員工: Robert Chen, 年齡: 38, 部門: IT
*/

// 展示不同的立即執行方法
Console.WriteLine("\n=== 展示立即執行方法 ===");

int count = query.Count();
Console.WriteLine($"符合條件的員工數量: {count}");

Employee firstEmployee = query.FirstOrDefault();
Console.WriteLine($"第一個符合條件的員工: {firstEmployee?.Name}");

bool anyMatch = query.Any();
Console.WriteLine($"是否有符合條件的員工: {anyMatch}");

/* 輸出結果：
=== 展示立即執行方法 ===
符合條件的員工數量: 3
第一個符合條件的員工: John Smith
是否有符合條件的員工: True
*/

// 展示多次執行相同查詢的效果
// 要向DB查詢兩次
Console.WriteLine("\n=== 不好的做法：重複執行查詢 ===");
foreach (var employee in context.Employees
    .Where(e => e.Department == "IT" && e.Age > 30))
{
    Console.WriteLine($"第一次查詢找到: {employee.Name}");
}
foreach (var employee in context.Employees
    .Where(e => e.Department == "IT" && e.Age > 30))
{
    Console.WriteLine($"第二次查詢找到: {employee.Name}");
}

/* 輸出結果：
=== 不好的做法：重複執行查詢 ===
第一次查詢找到: John Smith
第一次查詢找到: Mary Johnson
第一次查詢找到: Robert Chen
第二次查詢找到: John Smith
第二次查詢找到: Mary Johnson
第二次查詢找到: Robert Chen
*/

// 展示較好的做法
Console.WriteLine("\n=== 好的做法：快取查詢結果 ===");
List<Employee> cachedResults = context.Employees
    .Where(e => e.Department == "IT" && e.Age > 30)
    .ToList();

foreach (var employee in cachedResults)
{
    Console.WriteLine($"第一次使用快取: {employee.Name}");
}
foreach (var employee in cachedResults)
{
    Console.WriteLine($"第二次使用快取: {employee.Name}");
}

/* 輸出結果：
=== 好的做法：快取查詢結果 ===
第一次使用快取: John Smith
第一次使用快取: Mary Johnson
第一次使用快取: Robert Chen
第二次使用快取: John Smith
第二次使用快取: Mary Johnson
第二次使用快取: Robert Chen
*/
```

## List<int> 與 context.Employees 的遍歷差異

1. List<int> 的 Where 查詢（IEnumerable）：
```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// 建立查詢但尚未執行
IEnumerable<int> evenQuery = numbers.Where(n => {
    Console.WriteLine($"檢查數字: {n}");
    return n % 2 == 0;
});

// 執行查詢 - 會逐一檢查每個元素
foreach (int num in evenQuery)
{
    Console.WriteLine($"找到偶數: {num}");
}

/* 輸出結果：
檢查數字: 1
檢查數字: 2
找到偶數: 2
檢查數字: 3
檢查數字: 4
找到偶數: 4
檢查數字: 5
*/
```

2. Entity Framework 的 Where 查詢（IQueryable）：
```csharp
// 建立查詢但尚未執行
IQueryable<Employee> query = context.Employees.Where(e => {
    Console.WriteLine($"檢查員工: {e.Name}");  // 這行實際上不會執行！
    return e.Department == "IT";
});

// 執行查詢 - 會一次性取回所有符合條件的資料
foreach (var employee in query)
{
    Console.WriteLine($"找到員工: {employee.Name}");
}

/* 輸出結果：
-- 首先執行完整的 SQL 查詢
SELECT [e].[Id], [e].[Name], [e].[Age], [e].[Department]
FROM [Employees] AS [e]
WHERE [e].[Department] = 'IT'

-- 然後直接輸出結果
找到員工: John
找到員工: Mary
找到員工: Robert
*/
```

主要差異：

1. IEnumerable（記憶體中的集合）：
    - 是真正的「延遲執行」和「逐一處理」
    - 可以在處理過程中執行自訂邏輯
    - 適合小型資料集
    - Lambda 表達式中的程式碼實際會被執行

2. IQueryable（資料庫查詢）：
    - Where 條件會轉換成 SQL WHERE 子句
    - 一次性執行完整的 SQL 查詢
    - 一次性取回所有符合條件的資料
    - 減少了資料庫往返
    - 適合大型資料集
    - Lambda 表達式只用於建構 SQL，內部的程式碼不會被執行
    
3. 簡單記憶法：
    - IEnumerable：在記憶體中一個一個處理（像逐個檢查箱子裡的物品）
    - IQueryable：在資料庫一次找出所有符合條件的資料（像用清單一次買齊所有東西）
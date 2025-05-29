---
title: _Select
tags: [LINQ]

---

# 語法應用

1. `List<int>` (IEnumerable):
```csharp
List<int> numbers = new List<int> { 1, 2, 3 };

IEnumerable<int> doubled = numbers.Select(x => x * 2);
IEnumerable<string> asStrings = numbers.Select(x => x.ToString());
```

2. `context.Employees` (IQueryable):
```csharp
IQueryable<string> names = context.Employees.Select(e => e.Name);

IQueryable<EmployeeDTO> dtos = context.Employees.Select(e => new EmployeeDTO {
    Name = e.Name,
    Salary = e.Salary
});
```

3. 使用索引：
```csharp
// 定義原始陣列
string[] fruits = new[] { "蘋果", "香蕉", "橘子" };

// 方法 1：使用 Select 帶索引的多載方法
IEnumerable<string> indexedFruits1 = fruits.Select((fruit, index) => $"{index + 1}. {fruit}");
// 結果: { "1. 蘋果", "2. 香蕉", "3. 橘子" }

// 方法 2：使用匿名類型
IEnumerable<(int Index, string Name)> indexedFruits2 = fruits.Select((fruit, index) => (index + 1, fruit));
// 結果: { (1, "蘋果"), (2, "香蕉"), (3, "橘子") }

// 在 Entity Framework 中使用索引
IQueryable<EmployeeDTO> dtos = context.Employees.Select((e, i) => new EmployeeDTO {
   RowNumber = i + 1,
   Name = e.Name,
   Salary = e.Salary
});

```

# 延遲執行

1. 使用 List<int> 作為資料來源
```csharp
public class ListExample
{
    public void DemonstrateListLazyEvaluation()
    {
        Console.WriteLine("===== List<int> 延遲執行示範 =====");
        
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        Console.WriteLine($"原始數列: {string.Join(", ", numbers)}");
        
        // 建立 Select 查詢，但尚未執行
        IEnumerable<int> query = numbers.Select(n => 
        {
            Console.WriteLine($"正在處理數字: {n}"); // 用來觀察執行時機
            return n * 2;
        });
        
        Console.WriteLine("\n查詢已建立，但尚未執行");
        Console.WriteLine("此時不會看到「正在處理數字」的訊息");
        
        // 實際執行查詢（透過迭代）
        Console.WriteLine("\n第一次迭代結果:");
        foreach (int item in query)
        {
            Console.WriteLine($"結果: {item}");
        }
        
        // 修改原始資料
        Console.WriteLine("\n修改原始資料");
        numbers.Add(6);
        
        // 再次執行查詢
        Console.WriteLine("\n第二次迭代結果 (注意新加入的數字也會被處理):");
        foreach (int item in query)
        {
            Console.WriteLine($"結果: {item}");
        }
    }
}

/* 執行結果：
===== List<int> 延遲執行示範 =====
原始數列: 1, 2, 3, 4, 5

查詢已建立，但尚未執行
此時不會看到「正在處理數字」的訊息

第一次迭代結果:
正在處理數字: 1
結果: 2
正在處理數字: 2
結果: 4
正在處理數字: 3
結果: 6
正在處理數字: 4
結果: 8
正在處理數字: 5
結果: 10

修改原始資料

第二次迭代結果 (注意新加入的數字也會被處理):
正在處理數字: 1
結果: 2
正在處理數字: 2
結果: 4
正在處理數字: 3
結果: 6
正在處理數字: 4
結果: 8
正在處理數字: 5
結果: 10
正在處理數字: 6
結果: 12
*/
```

2. 使用 Entity Framework DbSet<Employee> 作為資料來源
```csharp
public class DbContextExample
{
    private readonly YourDbContext context;
    
    public DbContextExample(YourDbContext context)
    {
        this.context = context;
    }
    
    public void DemonstrateEfLazyEvaluation()
    {
        Console.WriteLine("===== Entity Framework 延遲執行示範 =====");
        
        // 建立查詢，但還不會執行 SQL
        IQueryable<EmployeeDTO> query = context.Employees
            .Select(e => new EmployeeDTO
            {
                Name = e.Name,
                AnnualSalary = e.Salary * 12
            });
            
        Console.WriteLine("1. 查詢已建立");
        Console.WriteLine("生成的 SQL (尚未執行):");
        Console.WriteLine("SELECT Name, Salary * 12 AS AnnualSalary FROM Employees");
        
        // 加入篩選條件，仍然不會執行
        IQueryable<EmployeeDTO> filteredQuery = query.Where(e => e.AnnualSalary > 500000);
        
        Console.WriteLine("\n2. 篩選條件已加入");
        Console.WriteLine("更新後的 SQL (仍未執行):");
        Console.WriteLine("SELECT Name, Salary * 12 AS AnnualSalary FROM Employees WHERE (Salary * 12) > 500000");
        
        // 實際執行查詢（此時才會發送 SQL 到資料庫）
        List<EmployeeDTO> results = filteredQuery.ToList();
        
        Console.WriteLine("\n3. 查詢已執行，從資料庫讀取的結果:");
        foreach (EmployeeDTO item in results)
        {
            Console.WriteLine($"員工: {item.Name}, 年薪: {item.AnnualSalary:C}");
        }
    }
}

/* 執行結果：
===== Entity Framework 延遲執行示範 =====
1. 查詢已建立
生成的 SQL (尚未執行):
SELECT Name, Salary * 12 AS AnnualSalary FROM Employees

2. 篩選條件已加入
更新後的 SQL (仍未執行):
SELECT Name, Salary * 12 AS AnnualSalary FROM Employees WHERE (Salary * 12) > 500000

3. 查詢已執行，從資料庫讀取的結果:
員工: John Doe, 年薪: $600,000.00
員工: Jane Smith, 年薪: $720,000.00
員工: Robert Johnson, 年薪: $550,000.00
*/

public class EmployeeDTO
{
    public string Name { get; set; }
    public decimal AnnualSalary { get; set; }
}
```

3. List<int> 與 context.Employees 遍歷差異
```csharp
// 示範 1: List<int> 的遍歷特性 (In-Memory Collection)
public class ListTraversalExample
{
    public void DemonstrateListTraversal()
    {
        Console.WriteLine("===== List<int> 遍歷示範 =====");
        
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // 1. 多次遍歷
        IEnumerable<int> query = numbers
            .Where(n => 
            {
                Console.WriteLine($"執行 Where 過濾: {n}");
                return n > 2;
            })
            .Select(n =>
            {
                Console.WriteLine($"執行 Select 轉換: {n}");
                return n * 2;
            });

        Console.WriteLine("\n第一次遍歷:");
        foreach (int result in query)
        {
            Console.WriteLine($"最終結果: {result}");
        }

        Console.WriteLine("\n第二次遍歷:");
        foreach (int result in query)
        {
            Console.WriteLine($"最終結果: {result}");
        }
    }
}

/* 執行結果：
===== List<int> 遍歷示範 =====

第一次遍歷:
執行 Where 過濾: 1
執行 Where 過濾: 2
執行 Where 過濾: 3
執行 Select 轉換: 3
最終結果: 6
執行 Where 過濾: 4
執行 Select 轉換: 4
最終結果: 8
執行 Where 過濾: 5
執行 Select 轉換: 5
最終結果: 10

第二次遍歷:
執行 Where 過濾: 1
執行 Where 過濾: 2
執行 Where 過濾: 3
執行 Select 轉換: 3
最終結果: 6
執行 Where 過濾: 4
執行 Select 轉換: 4
最終結果: 8
執行 Where 過濾: 5
執行 Select 轉換: 5
最終結果: 10
*/

// 示範 2: DbSet<Employee> 的遍歷特性 (Database Query)
public class DbSetTraversalExample
{
    private readonly YourDbContext context;
    
    public DbSetTraversalExample(YourDbContext context)
    {
        this.context = context;
    }
    
    public void DemonstrateDbSetTraversal()
    {
        Console.WriteLine("===== DbSet<Employee> 遍歷示範 =====");
        
        // 1. 建立查詢但尚未執行
        IQueryable<EmployeeDTO> query = context.Employees
            .Where(e => e.Salary > 5000)
            .Select(e => new EmployeeDTO
            {
                Name = e.Name,
                AnnualSalary = e.Salary * 12
            });

        Console.WriteLine("生成的 SQL:");
        Console.WriteLine(@"
            SELECT e.Name, e.Salary * 12 as AnnualSalary
            FROM Employees e
            WHERE e.Salary > 5000");
            
        // 2. 第一次執行查詢
        Console.WriteLine("\n第一次查詢執行結果:");        
        // query 執行查詢 - 一次性取回所有符合條件的資料，再foreach
        foreach (EmployeeDTO emp in query)
        {
            Console.WriteLine($"員工: {emp.Name}, 年薪: {emp.AnnualSalary:C}");
        }
        
        // 3. 第二次執行查詢
        Console.WriteLine("\n第二次查詢執行結果:");
        // ToList() 執行查詢 - 一次性取回所有符合條件的資料
        List<EmployeeDTO> results2 = query.ToList();
        foreach (EmployeeDTO emp in results2)
        {
            Console.WriteLine($"員工: {emp.Name}, 年薪: {emp.AnnualSalary:C}");
        }
    }
}

/* 執行結果：
===== DbSet<Employee> 遍歷示範 =====
生成的 SQL:
SELECT e.Name, e.Salary * 12 as AnnualSalary
FROM Employees e
WHERE e.Salary > 5000

第一次查詢執行結果:
員工: John Doe, 年薪: $72,000.00
員工: Jane Smith, 年薪: $84,000.00

第二次查詢執行結果:
員工: John Doe, 年薪: $72,000.00
員工: Jane Smith, 年薪: $84,000.00
*/
```
## List 特性與行為
- **資料儲存**：資料已完整存在於記憶體中
- **運作機制**：
  - 採用 IEnumerable<T> 介面
  - 每次遍歷都重新執行所有運算（Where、Select 等）
  - 逐項處理資料（一次處理一個元素）
  - 原始資料的修改會反映在下次遍歷中
- **適用場景**：
  - 小型資料集處理
  - 需要在記憶體中進行複雜運算
- **優缺點**：
  - 優點：可執行任意的 .NET 程式碼，適合複雜的記憶體內運算
  - 缺點：所有資料都必須載入到記憶體中，無法利用資料庫的優化能力

## DbContext 特性與行為
- **資料儲存**：資料存在於資料庫中
- **運作機制**：
  - 採用 IQueryable<T> 介面
  - 查詢會被轉換為單一 SQL 語句
  - 只在呼叫 ToList()、FirstOrDefault() 等方法時才實際執行查詢
  - Where 和 Select 的運算邏輯會被轉換成 SQL  
- **適用場景**：
  - 大型資料集處理
  - 只需要遍歷一次
  - 需要最新的資料
  - 查詢邏輯可以完全轉換為 SQL
- **優缺點**：
  - 優點：不需要額外的記憶體來存儲完整結果，適合處理大量資料
  - 缺點：如果需要多次遍歷 (多次 foreach)，會重複執行資料庫查詢
    
# Exception

1. 基本的 Select 異常情況：
```csharp
// 1. 空集合的處理 - 不會拋出異常
List<int> numbers = new List<int>();
IEnumerable<int> result = numbers.Select(x => x * 2); // 不會拋出異常，只是返回空集合

// 2. null 集合的處理
List<int> nullList = null;
try
{
    var result = nullList.Select(x => x * 2); // 拋出: NullReferenceException
}
catch (NullReferenceException ex)
{
    // 處理空參考異常
}
```

2. Select 處理中的 null 值：
```csharp
List<string> names = new List<string> { "Tom", null, "Jerry" };

// 1. 直接存取可能為 null 的值
try
{
    var lengths = names.Select(name => name.Length); // 拋出: NullReferenceException
}
catch (NullReferenceException ex)
{
    // 處理 null 值異常
}

// 安全的處理方式
var safeLengths = names.Select(name => name?.Length ?? 0);
// 結果: { 3, 0, 5 }
```

3. Select 中的型別轉換：
```csharp
List<string> stringNumbers = new List<string> { "1", "two", "3" };

// 1. 不安全的轉換
try
{
    var numbers = stringNumbers.Select(int.Parse); // 拋出: FormatException
}
catch (FormatException ex)
{
    // 處理格式轉換異常
}

// 安全的轉換方式
var safeNumbers = stringNumbers.Select(s => 
{
    if (int.TryParse(s, out int result))
        return result;
    return 0; // 或其他預設值
});
// 結果: { 1, 0, 3 }
```

4. 當 context.Employees 沒有資料時
```csharp
// 1. 基本的 Select - 不會拋出異常
var emptyResult = context.Employees
    .Select(e => new
    {
        e.Name,
        e.Department
    });
// 結果: 空的 IQueryable，不會拋出異常

// 檢查是否有資料的方式
if (!emptyResult.Any())
{
    Console.WriteLine("沒有資料");
}

// 2. 轉換成 List - 不會拋出異常
var emptyList = context.Employees
    .Select(e => new
    {
        e.Name,
        e.Department
    })
    .ToList();
// 結果: 空的 List，不會拋出異常

// 3. 空集合後的操作 - 安全
var count = emptyList.Count();    // 結果: 0
var isEmpty = !emptyList.Any();   // 結果: true
```

但是，如果在空集合後使用某些聚合方法，可能會拋出異常：

```csharp
try
{
    // 這些操作會在空集合時拋出異常
    var firstItem = context.Employees
        .Select(e => e.Name)
        .First();  // InvalidOperationException: 序列不包含任何元素

    var singleItem = context.Employees
        .Select(e => e.Name)
        .Single(); // InvalidOperationException: 序列不包含任何元素
        
    var maxSalary = context.Employees
        .Select(e => e.Salary)
        .Max();    // InvalidOperationException: 序列不包含任何元素
}
catch (InvalidOperationException ex)
{
    // 處理異常
}

// 安全的寫法
var firstOrDefault = context.Employees
    .Select(e => e.Name)
    .FirstOrDefault();  // 結果: null

var singleOrDefault = context.Employees
    .Select(e => e.Name)
    .SingleOrDefault(); // 結果: null

var maxSalaryOrDefault = context.Employees
    .Select(e => e.Salary)
    .DefaultIfEmpty(0)
    .Max();            // 結果: 0
```

1. 集合操作：
   - 使用 Any() 檢查是否有資料
   - 使用 ...OrDefault() 方法避免異常
   - 使用 DefaultIfEmpty() 提供預設值
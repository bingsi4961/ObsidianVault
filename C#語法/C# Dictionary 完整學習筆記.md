---
title: 'C# Dictionary 完整學習筆記'
tags: ['C# 程式', LINQ]

---

# C# Dictionary 完整學習筆記

## 概述

`Dictionary<TKey, TValue>` 是 C# 中用來儲存**多筆 Key-Value 配對資料**的集合型別，類似於其他語言中的 Map 或 Hash Table。每個 Key 都是唯一的，可以透過 Key 快速查找對應的 Value。

---

## 基本概念

### Dictionary 的特點
- **多筆資料**：可以儲存多個 Key-Value 配對
- **Key 唯一性**：<mark style="background: #FFF3A3A6;">每個 Key 在 Dictionary 中都是唯一的</mark>
- **快速查找**：透過 Key 進行 O(1) 時間複雜度的查找
- **動態調整**：可以在執行時新增、修改、刪除資料

### 與其他集合型別的比較
```csharp
// List - 使用索引存取
List<string> nameList = new List<string> { "張三", "李四", "王五" };

// Array - 使用索引存取  
string[] nameArray = { "張三", "李四", "王五" };

// Dictionary - 使用 Key 存取
Dictionary<int, string> nameDict = new Dictionary<int, string>
{
    { 1, "張三" },
    { 2, "李四" },  
    { 3, "王五" }
};
```

---

## ToDictionary 方法

`ToDictionary` 是 LINQ 中的擴充方法，可以將 `IEnumerable<T>` 集合轉換成 `Dictionary<TKey, TValue>`。

### 基本語法
```csharp
// 四種重載形式
source.ToDictionary(keySelector)
source.ToDictionary(keySelector, valueSelector)
source.ToDictionary(keySelector, comparer)
source.ToDictionary(keySelector, valueSelector, comparer)
```

### 常見使用情境

#### 1. 物件集合轉換
```csharp
var employees = new List<Employee>
{
    new Employee { Id = 1, Name = "張三", Department = "IT" },
    new Employee { Id = 2, Name = "李四", Department = "HR" },
    new Employee { Id = 3, Name = "王五", Department = "IT" }
};

// 將 Id 作為 Key，整個物件作為 Value
var employeeDict = employees.ToDictionary(emp => emp.Id);
// 結果：Dictionary<int, Employee>

// 將 Id 作為 Key，Name 作為 Value
var nameDict = employees.ToDictionary(emp => emp.Id, emp => emp.Name);
// 結果：Dictionary<int, string>
```

#### 2. 字串集合轉換
```csharp
var fruits = new[] { "Apple", "Banana", "Cherry" };

// 以第一個字母作為 Key
var fruitDict = fruits.ToDictionary(fruit => fruit[0]);

// 以長度作為 Key，轉大寫作為 Value
var lengthDict = fruits.ToDictionary(fruit => fruit.Length, fruit => fruit.ToUpper());
```

#### 3. Web 開發應用
```csharp
// 在 MVC Controller 中常見用法
public ActionResult Index()
{
    var products = GetProducts();
    
    // 建立產品 ID 對應名稱的字典，供前端使用
    var productNames = products.ToDictionary(p => p.Id, p => p.Name);
    
    ViewBag.ProductNames = productNames;
    return View(products);
}
```

#### 4. 資料庫查詢優化
```csharp
// Entity Framework 查詢後建立字典加速查找
var userRoles = dbContext.UserRoles
    .Where(ur => userIds.Contains(ur.UserId))
    .ToDictionary(ur => ur.UserId, ur => ur.RoleName);

// 快速查詢
foreach (var userId in userIds)
{
    if (userRoles.TryGetValue(userId, out string roleName))
    {
        Console.WriteLine($"User {userId} has role: {roleName}");
    }
}
```

### ToDictionary 重要注意事項

#### Key 必須唯一
```csharp
var numbers = new[] { 1, 2, 2, 3 }; // 有重複的 2

// 這會拋出 ArgumentException
try
{
    var dict = numbers.ToDictionary(n => n);
}
catch (ArgumentException ex)
{
    Console.WriteLine("Key 重複了！");
}
```

#### 處理重複 Key 的方法
```csharp
// 方法一：使用 GroupBy 先分組
var safeDict = numbers.GroupBy(n => n)
    .ToDictionary(g => g.Key, g => g.Count());

// 方法二：使用 Distinct 去除重複
var uniqueDict = numbers.Distinct().ToDictionary(n => n);
```

#### 自訂比較器
```csharp
var words = new[] { "Apple", "apple", "APPLE" };

// 使用不區分大小寫的比較器
var caseInsensitiveDict = words.ToDictionary(
    w => w, 
    w => w.Length, 
    StringComparer.OrdinalIgnoreCase
);
```

---

## Dictionary 存取方式

### 1. 基本存取（索引器）
```csharp
var students = new Dictionary<int, string>
{
    { 1, "張三" },
    { 2, "李四" },
    { 3, "王五" }
};

// 讀取資料
string studentName = students[1]; // 取得 "張三"

// 修改資料
students[1] = "張小三";

// 新增資料
students[4] = "趙七";
```

⚠️ **注意**：使用不存在的 Key 會拋出 `KeyNotFoundException`

### 2. 安全存取（推薦）
```csharp
// 使用 TryGetValue 方法
if (students.TryGetValue(1, out string name))
{
    Console.WriteLine($"學生姓名：{name}");
}
else
{
    Console.WriteLine("找不到該學號的學生");
}

// Web 開發中的常用模式
public string GetUserRole(int userId)
{
    if (_userRoles.TryGetValue(userId, out string role))
    {
        return role;
    }
    return "Guest"; // 預設角色
}
```

### 3. 檢查存在性
```csharp
var config = new Dictionary<string, string>
{
    { "DatabaseConnection", "Server=localhost..." },
    { "ApiKey", "abc123..." }
};

// 檢查 Key 是否存在
if (config.ContainsKey("DatabaseConnection"))
{
    string connString = config["DatabaseConnection"];
}

// 檢查 Value 是否存在
if (config.ContainsValue("abc123..."))
{
    Console.WriteLine("找到該 API Key");
}
```

### 4. 遍歷資料

#### 遍歷 Key-Value 配對
```csharp
var scores = new Dictionary<string, int>
{
    { "Alice", 85 },
    { "Bob", 92 },
    { "Charlie", 78 }
};

foreach (var item in scores)
{
    Console.WriteLine($"學生：{item.Key}，分數：{item.Value}");
}

// 使用解構語法 (C# 7.0+)
foreach (var (name, score) in scores)
{
    Console.WriteLine($"學生：{name}，分數：{score}");
}
```

#### 只遍歷 Keys 或 Values
```csharp
// 只遍歷 Keys
foreach (string studentName in scores.Keys)
{
    Console.WriteLine($"學生姓名：{studentName}");
}

// 只遍歷 Values
foreach (int score in scores.Values)
{
    Console.WriteLine($"分數：{score}");
}
```

---

## 實務應用範例

### MVC 控制器中的應用
```csharp
public class OrderController : Controller
{
    private readonly Dictionary<string, decimal> _shippingRates = new Dictionary<string, decimal>
    {
        { "台北", 60 },
        { "台中", 80 },
        { "高雄", 100 }
    };

    [HttpPost]
    public ActionResult CalculateShipping(string city)
    {
        if (_shippingRates.TryGetValue(city, out decimal rate))
        {
            return Json(new { success = true, rate = rate });
        }
        
        return Json(new { success = false, message = "該城市不在配送範圍" });
    }
    
    public ActionResult GetAvailableCities()
    {
        var cities = _shippingRates.Keys.ToList();
        return Json(cities, JsonRequestBehavior.AllowGet);
    }
}
```

### 快取機制實作
```csharp
public class CacheService
{
    private readonly Dictionary<int, Product> _productCache;
    
    public CacheService(IEnumerable<Product> products)
    {
        _productCache = products.ToDictionary(p => p.Id);
    }
    
    public Product GetProduct(int id)
    {
        return _productCache.TryGetValue(id, out Product product) ? product : null;
    }
}
```

### 與 JavaScript 的搭配使用
```javascript
// 從 Controller 取得的 JSON 資料
var productPrices = { "蘋果": 30, "香蕉": 25, "橘子": 40 };

// JavaScript 存取方式
console.log(productPrices["蘋果"]); // 30
console.log(productPrices.香蕉);     // 25

// 遍歷資料
for (var product in productPrices) {
    console.log(product + ": " + productPrices[product]);
}
```

---

## 存取方式選擇指南

| 情境 | 推薦方法 | 原因 |
|------|----------|------|
| 確定 Key 存在 | `dictionary[key]` | 直接快速 |
| 不確定 Key 是否存在 | `TryGetValue` | 安全，避免例外 |
| 需要檢查存在性 | `ContainsKey` | 明確的語意 |
| 需要遍歷所有資料 | `foreach` | 支援多種遍歷方式 |
| 效能要求高的查找 | `TryGetValue` | 避免例外處理開銷 |

---

## 最佳實務建議

1. **優先使用 `TryGetValue`**：比起直接使用索引器更安全
2. **注意 Key 的唯一性**：使用 `ToDictionary` 時要確保 Key 不重複
3. **適當使用快取**：Dictionary 非常適合實作記憶體快取
4. **選擇合適的 Key 型別**：通常使用 `int`、`string` 或 `Guid`
5. **考慮執行緒安全性**：多執行緒環境下考慮使用 `ConcurrentDictionary`

Dictionary 是 C# 開發中非常重要且實用的資料結構，掌握好它的使用方式能大幅提升程式的效能和可讀性！
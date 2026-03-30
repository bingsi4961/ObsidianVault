---
date: 2026-03-30 17:09
title: C# Dictionary 完整學習筆記 (下篇)
aliases:
tags:
  - CSharp_語法
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[CSharp Dictionary 完整學習筆記 (上篇)|C# Dictionary 完整學習筆記 (上篇)]]

---
## 一、認識 ToDictionary：把集合「變」成 Dictionary

上篇我們學了如何操作 Dictionary，但在實務中，資料通常不是一開始就以 Dictionary 的形式存在——更常見的情況是：你先從資料庫撈到一個 `List<T>`，然後**把它轉成 Dictionary 以加速後續查詢**。

這個轉換工作，就是 `ToDictionary` 的用途。它是 LINQ 提供的擴充方法，可以把任何 `IEnumerable<T>` 集合轉換成 `Dictionary<TKey, TValue>`。

### 基本語法

`ToDictionary` 有四種重載形式，但你最常用到的是前兩種：

```csharp
// 只指定 keySelector：Value 是整個物件本身
source.ToDictionary(keySelector)

// 同時指定 keySelector 和 valueSelector：可以自訂 Value
source.ToDictionary(keySelector, valueSelector)

// 進階：加上自訂比較器（處理大小寫等問題）
source.ToDictionary(keySelector, comparer)
source.ToDictionary(keySelector, valueSelector, comparer)
```

---

### 常見使用情境

#### 情境一：物件集合轉換

這是最典型的用法。你有一個 `List<Employee>`，想快速根據員工 ID 取得員工物件：

```csharp
var employees = new List<Employee>
{
    new Employee { Id = 1, Name = "張三", Department = "IT" },
    new Employee { Id = 2, Name = "李四", Department = "HR" },
    new Employee { Id = 3, Name = "王五", Department = "IT" }
};

// 將 Id 作為 Key，整個員工物件作為 Value
// 結果型別是 Dictionary<int, Employee>
var employeeDict = employees.ToDictionary(emp => emp.Id);

// 如果只需要名字，可以同時指定 Value 的選取器
// 結果型別是 Dictionary<int, string>
var nameDict = employees.ToDictionary(emp => emp.Id, emp => emp.Name);
```

#### 情境二：資料庫查詢後建立索引（Portal / GTS 系統最實際的場景）

這是你在日常工作中最常遇到的場景。從資料庫撈完資料之後，立刻建立 Dictionary，後續的操作就能享受 O(1) 的查詢速度：

```csharp
var userIds = new List<int> { 1, 2, 3, 4, 5 };

// 從 Entity Framework 查詢後，立刻建立字典
var userRoles = dbContext.UserRoles
    .Where(ur => userIds.Contains(ur.UserId))
    .ToDictionary(ur => ur.UserId, ur => ur.RoleName);

// 後續查詢直接命中，不再掃描集合
foreach (var userId in userIds)
{
    if (userRoles.TryGetValue(userId, out string roleName))
    {
        Console.WriteLine($"User {userId} 的角色是：{roleName}");
    }
}
```

#### 情境三：字串集合的轉換

有時候你的來源不是物件集合，而是純字串陣列。`ToDictionary` 一樣適用，只是 keySelector 的寫法不同：

```csharp
var fruits = new[] { "Apple", "Banana", "Cherry" };

// 以第一個字母作為 Key，整個字串作為 Value
// 結果：{ 'A' -> "Apple", 'B' -> "Banana", 'C' -> "Cherry" }
var fruitDict = fruits.ToDictionary(fruit => fruit[0]);

// 以字串長度作為 Key，轉大寫後的字串作為 Value
// ⚠️ 注意："Apple" 和 "Mango" 都是 5 個字，若集合裡同時存在，就會重複 Key 爆炸
var lengthDict = fruits.ToDictionary(fruit => fruit.Length, fruit => fruit.ToUpper());
```

#### 情境四：結合 LINQ Where 縮小範圍再建立索引

有時候你只需要特定條件的子集合，可以先篩選再建立字典：

```csharp
// 只針對 IT 部門員工建立索引
var itEmployeeDict = employees
    .Where(emp => emp.Department == "IT")
    .ToDictionary(emp => emp.Id);
```

#### 情境五：不區分大小寫的字串 Key

這在處理 HTTP Header、設定檔名稱、使用者輸入時很實用：

```csharp
var configKeys = new[] { "ApiKey", "apikey", "APIKEY" };

// 加上 StringComparer.OrdinalIgnoreCase，大小寫不同的 Key 視為相同
// ⚠️ 注意：這代表 "ApiKey" 和 "apikey" 會被視為同一個 Key，
//    如果集合裡兩個都有，一樣會拋出重複 Key 的例外
var caseInsensitiveDict = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase)
{
    { "ApiKey", 1 }
};

// 之後查詢時，"apikey"、"APIKEY"、"ApiKey" 都能找到同一筆資料
bool found = caseInsensitiveDict.ContainsKey("apikey"); // true
```

---

## 二、ToDictionary 最重要的注意事項：Key 唯一性

這是新手最常踩的雷，我要特別提醒你。

**`ToDictionary` 遇到重複 Key 時，會直接拋出例外**：

```
System.ArgumentException: An item with the same key has already been added.
```

```csharp
var numbers = new[] { 1, 2, 2, 3 }; // 有兩個 2

// 這會拋出 ArgumentException，程式崩潰
var dict = numbers.ToDictionary(n => n); // ❌ 爆炸！
```

遇到這個情況，你需要先搞清楚「為什麼會重複」，再選擇對應的處理策略。

---

## 三、重複 Key 的完整處理策略

遇到重複 Key，有兩種本質上不同的情況，處理方式也不同。

### 情況一：Key 欄位本身重複（真正需要處理的問題）

```csharp
// Id 重複 — 這才是真正需要處理的情況
var employees = new List<Employee>
{
    new Employee { Id = 1, Name = "張三", Department = "IT" },
    new Employee { Id = 1, Name = "李四", Department = "HR" }, // Id 重複！
};
```

**策略 A：只保留第一筆（保留最早的）**

```csharp
var empDict = employees
    .GroupBy(emp => emp.Id)
    .ToDictionary(g => g.Key, g => g.First());
```

**策略 B：只保留最後一筆（以最新的覆蓋舊的）**

```csharp
var empDict = employees
    .GroupBy(emp => emp.Id)
    .ToDictionary(g => g.Key, g => g.Last());
```

**策略 C：保留全部，一個 Key 對應多筆資料**

這是業務邏輯真正允許一個 Key 對應多筆資料的情況（例如：一個部門有多名員工）：

```csharp
// Dictionary<int, List<Employee>> — 一個 Id 對應一個員工清單
var empDict = employees
    .GroupBy(emp => emp.Id)
    .ToDictionary(g => g.Key, g => g.ToList());

// 使用時：
var list = empDict[1]; // 取得 Id=1 的所有員工清單
```

### 情況二：Value 欄位重複（不需要處理）

這個觀念很重要：**Dictionary 只要求 Key 唯一，Value 重複完全沒問題**。

```csharp
// Department 的值都是 "IT" → 完全正常，不會報錯
// 因為 Key（Id）都不同：1、2、3
var empDict = employees.ToDictionary(emp => emp.Id); // ✅ 完全沒問題
```

但如果你想**以 Value 欄位（例如 Department）當作 Key**，那情況就不同了——它就變成了 Key，就必須處理重複：

```csharp
// 以 Department 為 Key → "IT" 出現兩次，爆炸！
var empDict = employees.ToDictionary(emp => emp.Department); // ❌

// 正確做法：一個部門對應多位員工
var empByDept = employees
    .GroupBy(emp => emp.Department)
    .ToDictionary(g => g.Key, g => g.ToList()); // ✅
```

### 快速判斷表

|情境|建議做法|
|---|---|
|Key 重複，業務上只需要一筆|`GroupBy` + `First()` 或 `Last()`|
|Key 重複，業務上需要全部資料|`GroupBy` + `ToList()`|
|Value 重複|不需要處理，Dictionary 允許|
|不確定資料乾不乾淨|用 `GroupBy` 先分組再轉換，最安全|

---

## 四、回顧：從效能優化角度看 ToDictionary

上篇說過 Dictionary 的 O(1) 查詢優勢，現在我們用一個完整的程式碼範例讓你看到「優化前 vs. 優化後」的對比：

```csharp
// =========== 優化前（效能差）===========
// 假設 updDescs4DB 有 1,000 筆，relatedDescs 也有 1,000 筆
// 整個迴圈最多需要執行 1,000 × 1,000 = 1,000,000 次比較
foreach (var vmDesc in relatedDescs)
{
    // 每次迴圈都要掃描整個 updDescs4DB 集合
    var entyDesc = updDescs4DB.FirstOrDefault(
        dbDesc => dbDesc.KeypartId == vmKp.Id &&
                  dbDesc.KpcolumnId == vmDesc.KPColumnId);

    if (entyDesc != null)
    {
        // 更新邏輯...
    }
}


// =========== 優化後（效能好）===========
// 步驟 1：先把相關資料篩出來，建立索引（只做一次，O(n)）
var descriptionDict = updDescs4DB
    .Where(dbDesc => dbDesc.KeypartId == vmKp.Id) // 先過濾，減少索引大小
    .ToDictionary(dbDesc => dbDesc.KpcolumnId);    // KpcolumnId 為 Key

// 步驟 2：迴圈裡直接查索引（每次 O(1)）
// 總共只需要：1,000 次建立索引 + 1,000 次查詢 = 2,000 次操作
foreach (var vmDesc in relatedDescs)
{
    if (descriptionDict.TryGetValue(vmDesc.KPColumnId, out var entyDesc))
    {
        // 更新邏輯...
    }
}
```

這個優化方式在你的日常工作中很常用到，下次看到「在迴圈裡用 `FirstOrDefault` 查詢集合」的程式碼，就應該想到這個改法。

---

## 五、實務應用：快取機制實作

Dictionary 非常適合用來實作**簡單的記憶體快取**。下面這個範例展示了如何在 Service 層用 Dictionary 快取資料庫查詢結果：

```csharp
public class ProductCacheService
{
    // 用 Dictionary 儲存產品資料，以 Id 為 Key
    private readonly Dictionary<int, Product> _productCache;

    // 建構子：一次性載入所有資料並建立索引
    public ProductCacheService(IEnumerable<Product> products)
    {
        _productCache = products.ToDictionary(p => p.Id);
    }

    // 之後每次查詢都是 O(1)，不需要再打資料庫
    public Product GetProduct(int id)
    {
        // 找不到時回傳 null，不會拋出例外
        return _productCache.TryGetValue(id, out Product product)
            ? product
            : null;
    }

    // 也可以提供預設值版本
    public Product GetProductOrDefault(int id, Product defaultProduct = null)
    {
        return _productCache.TryGetValue(id, out Product product)
            ? product
            : defaultProduct;
    }
}
```

> 【補充說明】記憶體快取在 Portal / GTS 這類企業系統中要小心使用：它只適合**資料不頻繁變動**的情況（例如產品分類、部門清單、角色設定）。如果資料會被其他使用者修改，快取可能讀到舊資料。需要更完善的快取機制，可以考慮 `IMemoryCache`（.NET Core 內建）或 Redis。

---

## 六、最佳實務總結

學完了這兩篇，讓我用五個核心建議幫你做個收尾：

**優先使用 `TryGetValue` 取代索引器**：比起用 `dictionary[key]` 直接取值更安全，能避免 `KeyNotFoundException` 讓程式崩潰。只有在邏輯上確保 Key 一定存在時才用索引器。

**使用 `ToDictionary` 前確認 Key 不重複**：如果不確定資料的乾淨程度，用 `GroupBy` 先分組再轉換是最安全的作法。

**善用 Dictionary 實作快取**：對於資料庫查詢結果、需要重複查詢的集合，先建立 Dictionary 再查詢，能大幅提升效能。

**選擇合適的 Key 型別**：通常優先選 `int`、`string` 或 `Guid`。Key 型別需要能正確計算 `GetHashCode()` 和 `Equals()`，自訂類別當 Key 時需要特別注意這點。

**多執行緒環境改用 `ConcurrentDictionary`**：普通的 `Dictionary<TKey, TValue>` 不是 thread-safe 的，在多執行緒環境（例如 async/await 的背景工作）同時讀寫同一個 Dictionary 可能發生資料競爭。這種情況請改用 `System.Collections.Concurrent.ConcurrentDictionary<TKey, TValue>`。
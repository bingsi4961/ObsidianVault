---
date: 2025-09-04 11:19
aliases:
tags:
  - EntityFramework_6
  - EntityFramework_Core
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
## 從 N+1 到高階查詢策略的深度探索

### 前言

這份筆記旨在深入探討 Entity Framework (EF) Core 的查詢效能。我們將從最經典的「N+1 查詢問題」出發，一步步揭開 LINQ 查詢背後的「延遲執行」機制，並深入了解 EF Core 如何像一位聰明的翻譯官，將我們的 C# 程式碼轉換為高效的 SQL 指令。這不僅僅是解決問題，更是要徹底理解「為什麼要這樣做」。

## 第 1 章：N+1 查詢問題 - 沉默的效能殺手

N+1 是所有 ORM (物件關聯對應) 框架中最常見、也最隱蔽的效能陷阱。它會在你不知不覺中，讓你的應用程式與資料庫之間產生大量不必要的來回溝通。

### 1.1. 問題的典型場景

假設我們要查詢所有有效的 CPU 規格 (`Performance_BaseLineCPUSpec`)，並同時列出每種規格底下所有有效的 CPU 型號 (`Performance_BaseLineCPU`)。

```csharp
// 典型的 N+1 問題程式碼
var result = entities.Performance_BaseLineCPUSpec
    .Where(s => s.IsValid)
    .Select(s => new {
        Id = s.Id,
        Year = s.Year,
        // 問題根源：在 Select 投影中，對關聯的「集合」進行操作
        BaseLineCPUs = s.Performance_BaseLineCPU.Where(c => c.IsValid).ToList()
    })
    .ToList();
```

### 1.2. 運作原理的逐步分解

這段程式碼會導致 `1 + N` 次的資料庫查詢：

1. **第 "1" 次查詢**：外層的 `.ToList()` 像是一個啟動按鈕，觸發了主查詢。EF Core 會先撈回所有 `IsValid` 的 `Performance_BaseLineCPUSpec`。   

```sql
-- 這是那 "1" 次查詢
SELECT * FROM [Performance_BaseLineCPUSpec] WHERE [IsValid] = 1
```

假設這次查詢撈回了 **N** 筆規格資料（例如 N=10）。   
    
2. **第 "+N" 次查詢**：接下來，程式會在 C# 的記憶體中遍歷這 10 筆規格資料。對於**每一筆**規格，當程式執行到 `s.Performance_BaseLineCPU.Where(...)` 時，由於關聯的 CPU 資料先前並未被載入，EF Core 會「貼心地」為其發送一次**新的**、獨立的 SQL 查詢來取得資料。
    
```sql
-- 為第 1 筆規格查詢 CPU
SELECT * FROM [Performance_BaseLineCPU] WHERE [BaseLineCPUSpecId] = @Id_1 AND [IsValid] = 1
-- 為第 2 筆規格查詢 CPU
SELECT * FROM [Performance_BaseLineCPU] WHERE [BaseLineCPUSpecId] = @Id_2 AND [IsValid] = 1
-- ... 這個過程會重複 N 次 ...
```

**最終結果**：總共產生了 `1 (主查詢) + 10 (子查詢) = 11` 次資料庫查詢。

### 1.3. 生活中的比喻：去圖書館借書

- **N+1 的做法 (效率差)**：你先跟館員要了 10 本書的清單（第 1 次溝通）。然後，你拿著清單上的第一本書名去問館員：「這本書的作者是誰？」（第 2 次溝通）。接著再問第二本、第三本... 你總共跟館員溝通了 11 次。
    
- **有效率的做法**：你直接跟館員說：「請幫我找出這 10 本書，並**順便**把每本書的作者資訊也一併給我。」（總共**只有 1 次**溝通）。
    

## 第 2 章：解決方案 - 使用 `Include` 預先載入

要像有效率的借書方式一樣，我們需要使用「預先載入」(Eager Loading) 策略，明確告訴 EF Core 我們需要哪些關聯資料。

### 2.1. 修正後的程式碼

```csharp
var result = entities.Performance_BaseLineCPUSpec
                    .Where(s => s.IsValid)
                    .Include(s => s.Performance_BaseLineCPU) // 關鍵：預先載入關聯集合
                    .Select(s => new {
                        Id = s.Id,
                        Year = s.Year,
                        // 此時 s.Performance_BaseLineCPU 資料已在記憶體中
                        // 下面的 .Where 是在記憶體中進行過濾，不會再查詢資料庫
                        BaseLineCPUs = s.Performance_BaseLineCPU.Where(c => c.IsValid).ToList()
                    })
                    .ToList();
```

### 2.2. 產生的 SQL

現在，EF Core 只會產生一個使用 `LEFT JOIN` 的**單一查詢**，一次性取回主表和所有相關的子表資料。

```sql
SELECT s.*, c.*
FROM [Performance_BaseLineCPUSpec] AS s
LEFT JOIN [Performance_BaseLineCPU] AS c ON s.Id = c.BaseLineCPUSpecId
WHERE s.IsValid = 1
```

### 2.3. 進階技巧：過濾過的 Include (Filtered Include)

在 EF Core 5.0+，你甚至可以在 `Include` 中直接加入過濾條件，讓 SQL 更精準。

```csharp
var query = entities.Performance_BaseLineCPUSpec
                    // 在 Include 中直接加入 Where 條件
                    .Include(s => s.Performance_BaseLineCPU.Where(c => c.IsValid))
                    .Where(s => s.IsValid);
```

這會將過濾條件 `c.IsValid` 直接加到 `JOIN` 的 `ON` 子句中，從資料庫源頭就減少了資料傳輸量。

## 第 3 章：LINQ 的核心 - 延遲執行 vs. 立即執行

要真正理解為何 N+1 會發生，必須先了解 LINQ 的「延遲執行」(Deferred Execution) 機制。

### 3.1. `IQueryable`：一份尚未執行的作戰計畫

當你撰寫 LINQ to Entities 查詢時，你並不是在馬上執行它，而是在建立一個 `IQueryable` 物件。你可以把它想像成一份詳細的「作戰計畫」。

```csharp
// 步驟 1: 建立計畫的第一部分 (目標是誰)
var queryPlan = context.Products.Where(p => p.IsActive);

// 步驟 2: 擴充計畫 (要如何呈現)
queryPlan = queryPlan.Select(p => new { p.Name, p.Price });
```

此時，`queryPlan` 變數裡裝的只是一份計畫，還沒有任何 SQL 被送到資料庫。

### 3.2. 觸發執行的將軍們

只有當「執行指令」出現 <mark style="background: #FFF3A3A6;">並且在最外層</mark> 時，這份計畫才會被 EF Core 翻譯成 SQL 並送出。常見的執行指令有：

- `.ToList()`, `.ToArray()`: 將結果轉換為集合。
    
- `.FirstOrDefault()`, `.Single()`, `.First()`: 只取一筆結果。
    
- `.Count()`, `.Any()`, `.Max()`: 進行彙總計算。
    
- `foreach` 迴圈：當程式需要遍歷查詢結果時。
    
### 3.3. 容器的選擇：`.ToList()` vs. `.ToArray()`

兩者都會觸發執行，但回傳的容器不同：

- **`Array` (陣列)**：像一個**固定尺寸的置物櫃**。大小固定，無法新增或刪除。
    
- **`List` (列表)**：像一個**可伸縮的購物袋**。大小可變，提供 `.Add()`, `.Remove()` 等彈性操作。
    

> 在實務上，由於經常需要對查詢結果進行後續加工，`.ToList()` 的使用頻率遠高於 `.ToArray()`。

## 第 4 章：EF Core 翻譯官的智慧

EF Core 的翻譯能力，決定了什麼樣的 LINQ 寫法會高效，什麼樣的會產生問題。

### 4.1. 關鍵區別：存取單一物件 vs. 操作集合

這是理解 N+1 的核心所在。

| **比較項目**   | **存取單一物件 (o.Customer.Name)**           | **操作集合 (s.Items.ToList())**   |
| ---------- | -------------------------------------- | ----------------------------- |
| **操作類型**   | 單純的導覽屬性存取 (Navigation Property Access) | 對導覽**集合**進行方法呼叫 (Method Call) |
| **EF 的理解** | 「這是一個可以直接對應到資料庫欄位的路徑。」                 | 「這是一個需要在 C# 程式中執行的複雜操作。」      |
| **翻譯結果**   | 可以被翻譯成 SQL **JOIN**                    | 無法直接翻譯，觸發延遲載入 (Lazy Loading)  |
| **效能結果**   | **高效 (1 次查詢)**                         | **低效 (N+1 次查詢)**              |

> 🚨 **核心經驗法則**：
> 在 `Select` 投影中，如果需要存取關聯的**單一物件**屬性 (`Order.Customer`)，EF 通常能自動處理。
> 但如果需要存取關聯的**集合**屬性 (`Category.Products`)，<mark style="background: #FFF3A3A6;">就必須明確使用 Include</mark>。

### 4.2. 伺服器端函式：將工作外包給資料庫

在 <mark style="background: #FFF3A3A6;">Select 投影中</mark> 使用 EF Core 認識的彙總函式，**不會**造成 N+1，因為 EF Core 會把它們翻譯成 SQL 子查詢，讓資料庫直接完成計算。

- `c.Products.Count()` -> 翻譯成 `(SELECT COUNT(*) ...)`
    
- `c.Products.Any()` -> 翻譯成 `EXISTS (SELECT 1 ...)`
    
- `c.Products.Max(p => p.Price)` -> 翻譯成 `(SELECT MAX(Price) ...)`
    
#### 伺服器端 vs. 客戶端計算的陷阱

- **`c.Products.Count()` (伺服器端)**：高效。EF Core 將計數工作交給資料庫，只傳回一個數字。
    
- **`c.Products.ToList().Count` (客戶端)**：**災難**。`.ToList()` 會先把所有 `Product` 物件從資料庫撈回來，然後 C# 程式才在記憶體中去數有幾個。這就是 N+1。
    
## 第 5 章：高階 SQL 翻譯策略

當我們在 `Select` 中使用 `FirstOrDefault()` 來取得集合中的第一筆資料時，EF Core 會動用更高階的 SQL 技巧。

```csharp
// 取得每個分類下的第一件商品
var query = context.Categories.Select(c => new {
    CategoryName = c.Name,
    FirstProduct = c.Products.FirstOrDefault()
});
```

### 5.1. 策略一：`OUTER APPLY` (SQL Server)

`OUTER APPLY` 就像一個在資料庫內部執行的 `foreach` 迴圈，為每一筆主資料高效地執行子查詢。

```sql
SELECT c.*, p.*
FROM [Categories] AS c
OUTER APPLY (
    SELECT TOP 1 * FROM [Products] AS p WHERE p.CategoryId = c.Id ORDER BY p.Id
) AS p
```

`OUTER APPLY` 就像一個高效的迴圈，為每一筆主資料執行子查詢。

**比喻**：資料庫直接回傳一個**已經組裝好的樂高模型**。

### 5.2. 策略二：`FOR JSON PATH` (較新的 SQL Server)

這個技巧將子查詢的多個欄位打包成一個**單一的 JSON 文字字串**。

```sql
SELECT c.Id, c.Name,
    (
        SELECT TOP 1 * FROM [Products] AS p WHERE p.CategoryId = c.Id ORDER BY p.Id
        FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
    ) AS [FirstProductJson]
FROM [Categories] AS c
```

這個技巧將子查詢的多個欄位打包成一個單一的 JSON 文字字串 ( {"Name":"Chai","Price":18.00} )，符合 SQL 語法。
EF Core 在收到這個字串後，會在 C# 端將其反序列化還原成物件。

**比喻**：資料庫回傳一個**未組裝的樂高零件包**和說明書。EF Core 在收到後，需要在 C# 端將其**反序列化**（組裝）還原成物件。

### 5.3. 具現化 (Materialization)

無論 EF Core 在背後使用哪種 SQL 策略，它都會忠實地完成我們在 `Select` 中定義的「設計藍圖」。從 SQL 的原始結果，轉換成我們想要的 C# 物件形狀的過程，就叫做「具現化」。這也是 ORM 的強大之處：它隱藏了底層的複雜性，讓我們能用一致的方式操作物件。

```csharp
// 假設 "context" 是你的資料庫上下文 (DbContext)
var categoryRepresentations = context.Categories
    .Select(c => new {
        // 投影 Category 的基本資料
        CategoryId = c.Id,
        CategoryName = c.Name,
        // 核心：使用 FirstOrDefault 取得代表商品
        NewestProduct = c.Products.FirstOrDefault()
    })
    .ToList(); // 執行查詢


// --- 如何使用取到的值 ---
// ★★ 不管是策略一還是策略二的 SQL語法，EF Core 都會還原 NewestProduct 的物件，所以資料的存取方式都是一樣的
foreach (var item in categoryRepresentations)
{
    Console.WriteLine($"分類: {item.CategoryName}");

    // ⭐️ 關鍵：使用前一定要檢查是否為 null
    if (item.NewestProduct != null)
    {
        Console.WriteLine($"  => 最新商品: {item.NewestProduct.Name} (價格: {item.NewestProduct.Price})");
    }
    else
    {
        Console.WriteLine("  => (此分類下尚無任何商品)");
    }
    Console.WriteLine("---");
}
```

```sql
-- 這是由上面的 LINQ 查詢所產生的高效能 SQL
SELECT
    [c].[Id] AS [CategoryId],
    [c].[Name] AS [CategoryName],
    [p].[Id],
    [p].[Name],
    [p].[Price],
    [p].[CategoryId] AS [ProductCategoryId] -- Product 的所有欄位
FROM [Categories] AS [c]
OUTER APPLY (
    -- 這個子查詢會為外面每一筆的分類 'c' 執行一次
    SELECT TOP 1 *
    FROM [Products] AS [p]
    WHERE [p].[CategoryId] = [c].[Id]
) AS [p]
```

## 第 6 章：導覽屬性的重要性

### 6.1. 有導覽屬性 (最佳實踐)

在資料模型中明確定義關聯，讓 EF Core 有清晰的「路標」。

```csharp
// 寫法清晰，直接表達物件關聯
FirstModel = g.ModelSettings.FirstOrDefault()
```

EF Core 會直接理解並產生高效的單一查詢。

### 6.2. 無導覽屬性 (應避免)

手動在 `Where` 條件中建立關聯。

```csharp
FirstModel = entities.ModelSettings.Where(m => m.GroupId == g.Id).FirstOrDefault()
```

> **結論**：雖然現代 EF Core 非常聰明，能分析 `Where` 條件並**推斷**出關聯，<mark style="background: #FFF3A3A6;">進而產生高效的單一查詢</mark>。但**永遠建議在資料模型中明確定義 ForeignKey 和導覽屬性**。這會讓程式碼更清晰、語意更明確，也最能發揮 ORM 的優勢。
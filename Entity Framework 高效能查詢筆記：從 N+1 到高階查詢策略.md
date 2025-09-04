---
date : 2025-09-04 10:34
aliases:
  - 別名測試1
  - 別名測試2
tags:
  - 標籤測試1
  - 標籤測試2

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

<style> .styled-table { border-collapse: collapse; margin: 25px 0; font-size: 0.9em; font-family: sans-serif; min-width: 400px; box-shadow: 0 0 20px rgba(0, 0, 0, 0.15); border-radius: 5px; overflow: hidden; } .styled-table thead tr { background-color: #009879; color: #ffffff; text-align: left; } .styled-table th, .styled-table td { padding: 12px 15px; text-align: left; } .styled-table tbody tr { border-bottom: 1px solid #dddddd; } .styled-table tbody tr:nth-of-type(even) { background-color: #f3f3f3; } .styled-table tbody tr:last-of-type { border-bottom: 2px solid #009879; } blockquote { border-left: 5px solid #009879; padding-left: 15px; color: #666; background-color: #f9f9f9; } </style>

# Entity Framework 高效能查詢筆記：從 N+1 到高階查詢策略

這份筆記整理了在使用 Entity Framework (EF) Core 時，從最常見的 N+1 效能問題，到 LINQ 查詢背後的執行原理，以及 EF Core 如何將 C# 程式碼智慧地翻譯成高效 SQL 的完整過程。

## 1. 核心問題：N+1 查詢 (The N+1 Query Problem)

N+1 是使用 ORM 框架時最常見的效能殺手。

### 問題範例

```
var query = entities.Performance_BaseLineCPUSpec.Where(s => s.IsValid);
var result = query.Select(s => new ATWebViewModel.BaseLineCPUSpecModel {
    Id = s.Id,
    Year = s.Year,
    // 問題根源：在投影中對「集合」進行操作
    BaseLineCPUs = s.Performance_BaseLineCPU.Where(c => c.IsValid).ToList()
}).ToList();
```

### 運作原理

1. **第 "1" 次查詢**：外層的 `.ToList()` 觸發第一次查詢，撈回所有 `Performance_BaseLineCPUSpec` 主資料。假設撈回了 **N** 筆。
    
    ```
    SELECT * FROM [Performance_BaseLineCPUSpec] WHERE [IsValid] = 1
    ```
    
2. **第 "+N" 次查詢**：程式在記憶體中處理這 N 筆主資料。對於**每一筆**主資料，當它要取得 `BaseLineCPUs` 這個集合時，EF 會為其發送一次**新的**獨立查詢。
    
    ```
    -- 為第一筆主資料查詢
    SELECT * FROM [Performance_BaseLineCPU] WHERE [BaseLineCPUSpecId] = @Id_1
    -- 為第二筆主資料查詢
    SELECT * FROM [Performance_BaseLineCPU] WHERE [BaseLineCPUSpecId] = @Id_2
    -- ... 重複 N 次 ...
    ```
    

總共產生 `1 + N` 次資料庫查詢，造成嚴重效能瓶頸。

## 2. 基礎解法：使用 `Include` 預先載入

要解決 N+1，必須使用「預先載入」(Eager Loading) 的策略，告訴 EF Core 一次性地把需要的關聯資料全部撈回來。

### 修正後的程式碼

```
var query = entities.Performance_BaseLineCPUSpec
                    .Include(s => s.Performance_BaseLineCPU) // 關鍵：預先載入關聯集合
                    .Where(s => s.IsValid);

var result = query.Select(s => new ATWebViewModel.BaseLineCPUSpecModel {
    Id = s.Id,
    Year = s.Year,
    // 此處的 .Where 是在記憶體中進行過濾，不會再查詢資料庫
    BaseLineCPUs = s.Performance_BaseLineCPU.Where(c => c.IsValid).ToList()
}).ToList();
```

### 產生的 SQL

EF Core 會產生一個使用 `LEFT JOIN` 的**單一查詢**，一次性取回所有需要的資料。

```
SELECT s.*, c.*
FROM [Performance_BaseLineCPUSpec] AS s
LEFT JOIN [Performance_BaseLineCPU] AS c ON s.Id = c.BaseLineCPUSpecId
WHERE s.IsValid = 1
```

## 3. LINQ 的智慧：投影 vs. 延遲執行

### 3.1. 為何 `o.Customer.Name` 不會 N+1？

EF Core 足夠聰明，能夠區分「存取單一物件」和「操作集合」。

<table class="styled-table"> <thead> <tr> <th>比較項目</th> <th>存取單一物件 (o.Customer.Name)</th> <th>操作集合 (s.Items.ToList())</th> </tr> </thead> <tbody> <tr> <td><strong>操作類型</strong></td> <td>單純的導覽屬性存取</td> <td>對導覽<strong>集合</strong>進行方法呼叫</td> </tr> <tr> <td><strong>EF 的理解</strong></td> <td>「這是一個可以直接對應到資料庫欄位的路徑。」</td> <td>「這是一個需要在 C# 程式中執行的複雜操作。」</td> </tr> <tr> <td><strong>翻譯結果</strong></td> <td>可以被翻譯成 SQL <strong>JOIN</strong></td> <td>無法直接翻譯，觸發延遲載入 (Lazy Loading)</td> </tr> <tr> <td><strong>效能結果</strong></td> <td><strong>高效 (1 次查詢)</strong></td> <td><strong>低效 (N+1 次查詢)</strong></td> </tr> </tbody> </table>

> **經驗法則**：在 `Select` 投影中，如果需要存取關聯的**單一物件**屬性，EF 通常能自動處理。但如果需要存取關聯的**集合**屬性，就必須明確使用 `Include` 來預先載入資料。

### 3.2. 延遲執行 (Deferred Execution)

LINQ 查詢在遇到「執行指令」前，都只是一份「查詢計畫」(`IQueryable`)。

- **建立計畫**：`Where()`, `Select()`, `OrderBy()` 等方法只是在組合查詢計畫，不會真的執行。
    
- **觸發執行**：`.ToList()`, `.ToArray()`, `.FirstOrDefault()`, `.Count()`, `foreach` 迴圈等，會觸發計畫的執行，將其翻譯成 SQL 送到資料庫。
    

## 4. `Select` 投影中的高階函式

在 `Select` 投影中使用 `Count()`, `Any()`, `FirstOrDefault()` 等函式，**不會**造成 N+1 問題，因為 EF Core 會將它們翻譯成高效的 SQL 子查詢。

### 範例：`FirstOrDefault()`

```
var categoryRepresentations = context.Categories
    .Select(c => new {
        CategoryName = c.Name,
        // 先排序再取第一個
        NewestProduct = c.Products
                         .OrderByDescending(p => p.Id)
                         .FirstOrDefault()
    })
    .ToList();

// 使用時務必檢查 null
foreach (var item in categoryRepresentations)
{
    if (item.NewestProduct != null) { /* ... */ }
}
```

### `FirstOrDefault()` 的 SQL 翻譯策略

EF Core 會使用以下兩種高效的 SQL 策略之一來完成任務：

1. **`OUTER APPLY`** (SQL Server)
    
    ```
    SELECT c.*, p.*
    FROM [Categories] AS c
    OUTER APPLY (
        SELECT TOP 1 *
        FROM [Products] AS p
        WHERE p.CategoryId = c.Id
        ORDER BY p.Id DESC
    ) AS p
    ```
    
    `OUTER APPLY` 就像一個高效的迴圈，為每一筆主資料執行子查詢。
    
2. **`FOR JSON PATH`** (較新的 SQL Server)
    
    ```
    SELECT
        c.Id, c.Name,
        (
            SELECT TOP 1 *
            FROM [Products] AS p
            WHERE p.CategoryId = c.Id
            ORDER BY p.Id DESC
            FOR JSON PATH, WITHOUT_ARRAY_WRAPPER
        ) AS [NewestProductJson]
    FROM [Categories] AS c
    ```
    
    這個技巧將子查詢的多個欄位打包成一個**單一的 JSON 文字字串**，符合 SQL 語法。EF Core 在收到這個字串後，會在 C# 端將其**反序列化**還原成物件。
    

## 5. 無導覽屬性的情況

即使資料模型沒有定義導覽屬性，現代 EF Core 依然很聰明。

```
// 手動在 Where 條件中建立關聯
var result = entities.GroupSetting
    .Select(g => new {
        GroupId = g.Id,        
        FirstModel = entities.ModelSettings
                               .Where(m => m.GroupId == g.Id)
                               .FirstOrDefault()
    })
    .ToList();
```

> **結論**：現代 EF Core 會分析 `Where(m => m.GroupId == g.Id)` 這個條件，**推斷**出兩者的關聯，並盡力產生與有導覽屬性時相同的高效單一查詢 (如 `OUTER APPLY`)。
> 
> **最佳實踐**：儘管 EF Core 很強大，但**永遠建議在資料模型中明確定義 ForeignKey 和導覽屬性**。這會讓程式碼更清晰、語意更明確，也最能發揮 ORM 的優勢。
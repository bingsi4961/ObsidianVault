---
title: GroupBy
tags: [LINQ]

---

## 1. GroupBy 基本概念

GroupBy 是 LINQ (Language Integrated Query) 中的一個關鍵方法，用於將集合分組處理。它能依照指定的條件將資料集合分割成多個子集合，每個子集合擁有相同的分組鍵值。

### 1.1 基本語法

```csharp
var groups = collection.GroupBy(item => item.PropertyName);

// 遍歷分組
foreach (var group in groups)
{
    Console.WriteLine($"Key: {group.Key}");
    foreach (var item in group)
    {
        Console.WriteLine($"  Item: {item}");
    }
}
```

### 1.2 核心元素與結構關係

```
原始集合 (collection)
[
    { Category: "電子", Name: "手機", Price: 15000 },
    { Category: "電子", Name: "電腦", Price: 30000 },
    { Category: "家具", Name: "桌子", Price: 8000 },
    { Category: "電子", Name: "耳機", Price: 2000 },
    { Category: "家具", Name: "椅子", Price: 4000 }
]

↓ .GroupBy(item => item.Category)

groups (IEnumerable<IGrouping<TKey, TElement>>)
┌─────────────────────────────────────────────┐
│                                             │
│  ┌─────────────┐        ┌─────────────┐     │
│  │   group 1   │        │   group 2   │     │
│  │ Key: "電子"  │        │ Key: "家具"  │     │
│  │             │        │             │     │
│  │  [手機]      │        │  [桌子]      │     │
│  │  [電腦]      │        │  [椅子]      │     │
│  │  [耳機]      │        │             │     │
│  └─────────────┘        └─────────────┘     │
│                                             │
└─────────────────────────────────────────────┘
```

- **collection**：原始資料集合
- **groups**：分組後的結果，類型為 `IEnumerable<IGrouping<TKey, TElement>>`
- **group**：每個分組，類型為 `IGrouping<TKey, TElement>`
    - 每個 group 既是鍵值的容器，也是該鍵值對應元素的集合
- 當您遍歷 groups 時，您是在遍歷不同的分組；而當您遍歷特定 group 時，您是在遍歷該分組中的元素。
- **Key**：識別分組的鍵值（例如："電子"、"家具"）
- **元素集合**：屬於該分組的所有原始元素

## 2. IGrouping 介面詳解

`IGrouping<TKey, TElement>` 是分組操作的核心介面，具有雙重角色：

### 2.1 介面定義

```csharp
public interface IGrouping<out TKey, out TElement> : IEnumerable<TElement>, IEnumerable
{
    TKey Key { get; }
}
```

### 2.2 IGrouping 特性

- 它同時是一個集合（可以遍歷其中的元素）和一個鍵值容器（可以訪問其 Key 屬性）
- **集合角色**：繼承自 `IEnumerable<TElement>`，可以直接遍歷其中的元素
- **鍵值容器**：透過 `Key` 屬性提供對分組鍵的存取
- **唯讀性質**：無法添加或移除元素
- **延遲計算**：只有在實際使用時才執行分組操作

### 2.3 使用範例

```csharp
// 取得 IGrouping 對象
IEnumerable<IGrouping<string, Product>> groupedProducts = products.GroupBy(p => p.Category);

// 遍歷所有分組
foreach (IGrouping<string, Product> group in groupedProducts)
{
    Console.WriteLine($"類別：{group.Key}");
    
    // 計算分組的統計資料
    decimal totalPrice = group.Sum(p => p.Price);
    decimal averagePrice = group.Average(p => p.Price);
    int count = group.Count();
    
    Console.WriteLine($"  產品數量：{count}");
    Console.WriteLine($"  總價值：{totalPrice}");
    Console.WriteLine($"  平均價格：{averagePrice}");
    
    // 遍歷分組中的元素
    foreach (Product product in group)
    {
        Console.WriteLine($"  - {product.Name}: {product.Price}");
    }
}
```

## 3. GroupBy 重載方法全解析

### 3.1 重載方法總覽

| 重載 | 參數 | 功能說明 |
|------|------|----------|
| 1 | keySelector | 基本分組功能 |
| 2 | keySelector, comparer | 使用指定的比較器進行分組 |
| 3 | keySelector, elementSelector | 轉換分組中的元素 |
| 4 | keySelector, elementSelector, comparer | 結合元素轉換與鍵比較 |
| 5 | keySelector, resultSelector | 直接將分組轉換為結果 |
| 6 | keySelector, elementSelector, resultSelector | 結合元素轉換與結果轉換 |
| 7 | keySelector, elementSelector, resultSelector, comparer | 最完整的功能組合 |

### 3.2 重載方法詳解

#### (1) 基本重載 - 僅指定鍵選擇器

```csharp
var groupedByCategory = products.GroupBy(p => p.Category);
```
這裡 p => p.Category 是鍵選擇器，它從每個產品中提取類別作為分組的鍵。

#### (2) 使用鍵比較器的重載

```csharp
// 範例 - 不區分大小寫分組
var caseInsensitiveGroups = products.GroupBy(
    p => p.Category,
    StringComparer.OrdinalIgnoreCase
);
```
這樣，"Electronics" 和 "electronics" 將被視為相同的類別。

#### (3) 使用元素選擇器的重載

```csharp
// 範例 - 只保留產品名稱
var groupedNames = products.GroupBy(
    p => p.Category,    // 鍵選擇器
    p => p.Name         // 元素選擇器
);

// 結果類型是 IEnumerable<IGrouping<string, string>>
// 每個分組的元素是產品名稱字串，而不是完整的產品對象
```
使用這個重載後，分組中的元素不再是原始的產品對象，而是經過轉換的名稱字串。

#### (4) 使用元素選擇器和鍵比較器的重載

```csharp
// 範例
var groupedNames = products.GroupBy(
    p => p.Category,                // 鍵選擇器
    p => p.Name,                    // 元素選擇器
    StringComparer.OrdinalIgnoreCase // 鍵比較器
);
```

#### (5) 使用結果選擇器的重載

```csharp
// 範例 - 直接計算分組統計
var categorySummaries = products.GroupBy(
    p => p.Category,                              // 鍵選擇器
    (key, groupItems) => new {                    // 結果選擇器 (IEnumerable<IGrouping<string,Product>>.Select())
        Category = key,
        Count = groupItems.Count(),
        TotalValue = groupItems.Sum(p => p.Price),
        AveragePrice = groupItems.Average(p => p.Price)
    }
);

// 結果類型是 IEnumerable<匿名類型>，每個元素包含分類摘要
```

#### (6) 使用元素選擇器和結果選擇器的重載

```csharp
// 範例 - 只使用價格進行統計
var priceStatistics = products.GroupBy(
    p => p.Category,                              // 鍵選擇器
    p => p.Price,                                 // 元素選擇器 - 只取價格
    (key, prices) => new {                        // 結果選擇器
        Category = key,
        TotalValue = prices.Sum(),
        AveragePrice = prices.Average()
    }
);
```

#### (7) 最完整的重載

```csharp
// 範例 - 完整功能組合
var completeExample = products.GroupBy(
    p => p.Category,                                // 鍵選擇器
    p => p.Price,                                   // 元素選擇器
    (key, prices) => new {                          // 結果選擇器
        Category = key,
        TotalValue = prices.Sum(),
        AveragePrice = prices.Average()
    },
    StringComparer.OrdinalIgnoreCase                // 鍵比較器
);
```
在此示例中，分組中的元素只是價格值，而不是完整的產品對象，然後直接計算統計數據。

## 4. 實用範例

### 4.1 按類別分組產品並計算各類別總價值

```csharp
List<Product> products = new List<Product>
{
    new Product { Name = "筆記型電腦", Category = "電子產品", Price = 30000 },
    new Product { Name = "手機", Category = "電子產品", Price = 15000 },
    new Product { Name = "桌子", Category = "傢俱", Price = 8000 },
    new Product { Name = "椅子", Category = "傢俱", Price = 4000 },
    new Product { Name = "平板電腦", Category = "電子產品", Price = 12000 }
};

// 按類別分組並計算每個類別的總價值
var categoryGroups = products
    .GroupBy(p => p.Category)
    .Select(g => new
    {
        Category = g.Key,
        TotalValue = g.Sum(p => p.Price),
        ProductCount = g.Count()
    });

// 顯示結果
foreach (var group in categoryGroups)
{
    Console.WriteLine($"類別: {group.Category}, 產品數量: {group.ProductCount}, 總價值: {group.TotalValue:C}");
}
```

### 4.2 按多重條件分組

```csharp
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public string CustomerRegion { get; set; }
    public string ProductCategory { get; set; }
    public decimal Amount { get; set; }
}

var regionCategoryGroups = orders
    .GroupBy(o => new { o.CustomerRegion, o.ProductCategory })
    .Select(g => new
    {
        Region = g.Key.CustomerRegion,
        Category = g.Key.ProductCategory,
        TotalAmount = g.Sum(o => o.Amount),
        OrderCount = g.Count()
    })
    .OrderByDescending(x => x.TotalAmount);;

// 顯示結果
foreach (var group in regionCategoryGroups)
{
    Console.WriteLine($"地區: {group.Region}, 類別: {group.Category}");
    Console.WriteLine($"  訂單數: {group.OrderCount}, 總金額: {group.TotalAmount:C}");
}
```

### 4.3 按時間週期分組

```csharp
// 按月份分組訂單
var ordersByMonth = orders
    .GroupBy(o => new { o.OrderDate.Year, o.OrderDate.Month })
    .Select(g => new
    {
        Year = g.Key.Year,
        Month = g.Key.Month,
        TotalAmount = g.Sum(o => o.Amount),
        OrderCount = g.Count()
    })
    .OrderBy(x => x.Year)
    .ThenBy(x => x.Month);

// 顯示結果
foreach (var group in ordersByMonth)
{
    Console.WriteLine($"{group.Year}年{group.Month}月: {group.OrderCount}筆訂單, 總額: {group.TotalAmount:C}");
}
```

### 4.4 結合其他 LINQ 方法

```csharp
// 找出每個類別中價格最高的產品
var highestPriceProducts = products
    .GroupBy(p => p.Category)
    .Select(g => new
    {
        Category = g.Key,
        MostExpensiveProduct = g.OrderByDescending(p => p.Price).First()
    });

// 計算各類別的平均價格，且只顯示平均價格超過10000的類別
var expensiveCategories = products
    .GroupBy(p => p.Category)
    .Select(g => new
    {
        Category = g.Key,
        AveragePrice = g.Average(p => p.Price)
    })
    .Where(x => x.AveragePrice > 10000)
    .OrderByDescending(x => x.AveragePrice);
```

## 5. 效能考量與最佳實踐

### 5.1 記憶體使用

- GroupBy 會在記憶體中保存分組的資料，大量資料可能導致記憶體消耗增加
- 考慮使用 `AsEnumerable()` 或 `AsQueryable()` 在適當的時候控制查詢執行位置

### 5.2 查詢效能

- 若需要多次訪問分組結果，考慮使用 `ToLookup()` 方法代替 `GroupBy()`
- 使用 `ToList()` 或 `ToDictionary()` 實體化結果，避免重複計算
- 在資料庫查詢中，盡量讓分組在資料庫層面完成，而非記憶體中

### 5.3 LINQ 鏈式操作最佳實踐

- 先使用 `Where` 篩選資料，再進行 `GroupBy` 操作
- 在 `GroupBy` 後使用 `Select` 立即轉換結果，減少中間結果的記憶體占用
- 對於複雜的分組邏輯，考慮使用多重 LINQ 查詢或分步處理

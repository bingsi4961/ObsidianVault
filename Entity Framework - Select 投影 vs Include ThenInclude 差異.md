---
date: 2025-07-23 09:10
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
#### 📑 [[]]

---
## 核心概念差異

### Select 投影 vs Include/ThenInclude 的根本差異

1. **回傳資料類型不同**
    - **Select 投影**: 回傳匿名物件或 DTO，無法存取原始實體
    - **Include/ThenInclude**: 回傳完整的實體物件，可存取所有導航屬性

### 程式碼範例對比

```csharp
// Select 投影
var orderData = context.Orders
    .Select(o => new {
        OrderId = o.Id,
        CustomerName = o.Customer.Name
    })
    .ToList();
// 結果：orderData[0].Customer 無法存取！

// Include/ThenInclude
var orders = context.Orders
    .Include(o => o.Customer)
    .ThenInclude(c => c.Address)
    .ToList();
// 結果：orders[0].Customer.Address.City 可以存取
```

## 使用情境選擇

### Select 投影適合的場景

- API 回傳特定資料
- 報表查詢
- 只需要部分欄位
- 唯讀操作

### Include/ThenInclude 適合的場景

- 需要完整實體物件
- 後續要更新資料
- 需要存取多個導航屬性
- 商業邏輯處理

## 實際應用範例

### 場景 1: API 回傳資料 (推薦 Select 投影)

```csharp
// ✅ Select 投影更適合
public IActionResult GetOrderList()
{
    var data = context.Orders
        .Select(o => new {
            o.Id,
            o.OrderDate,
            CustomerName = o.Customer.Name,
            CustomerCity = o.Customer.Address.City
        })
        .ToList();
    
    return Json(data); // 輕量、高效
}

// ❌ Include/ThenInclude 會載入不需要的資料
public IActionResult GetOrderListBad()
{
    var orders = context.Orders
        .Include(o => o.Customer)
        .ThenInclude(c => c.Address)
        .ToList();
    
    return Json(orders); // 會序列化所有屬性，包含不需要的
}
```

### 場景 2: 商業邏輯處理 (推薦 Include/ThenInclude)

```csharp
// ✅ Include/ThenInclude 更適合
public void ProcessOrder(int orderId)
{
    var order = context.Orders
        .Include(o => o.Customer)
        .ThenInclude(c => c.Address)
        .Include(o => o.OrderItems)
        .FirstOrDefault(o => o.Id == orderId);
    
    // 可以存取完整的實體物件
    order.Customer.LastOrderDate = DateTime.Now;
    order.OrderItems.Add(new OrderItem { ... });
    
    context.SaveChanges(); // 可以更新
}

// ❌ Select 投影無法用於更新
public void ProcessOrderBad(int orderId)
{
    var orderData = context.Orders
        .Select(o => new {
            o.Id,
            CustomerName = o.Customer.Name
        })
        .FirstOrDefault(o => o.Id == orderId);
    
    // 無法更新，因為不是實體物件
    // orderData.Customer.LastOrderDate = DateTime.Now; // 編譯錯誤！
}
```

## 效能與 SQL 差異

### 關鍵差異對比

|比較項目|Select 投影|Include/ThenInclude|
|---|---|---|
|**傳輸資料量**|只載入所需欄位|載入完整實體的所有欄位|
|**記憶體使用**|輕量|較重|
|**查詢效率**|高（精準）|中等（全面）|
|**網路傳輸**|少|多|

### SQL 產生差異示例

```sql
-- Select 投影：精簡查詢（只選擇需要的欄位）
SELECT [o].[Id], [c].[Name] AS [CustomerName]
FROM [Orders] AS [o] JOIN [Customers] AS [c] ON [o].[CustomerId] = [c].[Id]

-- Include/ThenInclude：完整查詢（載入所有欄位）
SELECT [o].[Id], [o].[OrderDate], [o].[Amount], [o].[CreatedDate], [o].[UpdatedDate],
       [c].[Id], [c].[Name], [c].[Email], [c].[Phone], [c].[CreatedDate], [c].[UpdatedDate],
       [a].[Id], [a].[Street], [a].[City], [a].[Country], [a].[PostalCode]
FROM [Orders] AS [o] JOIN [Customers] AS [c] ON ... JOIN [Addresses] AS [a] ON ...
```

## 系統版本差異重點

### Portal 系統 (EF Core 3.1.2)

- **語法特色**: 支援 `.ThenInclude()` 方法鏈
- **異步支援**: 完整的 `async/await`
- **推薦寫法**: 現代化語法

### GTS 系統 (EF6.1.3)

- **語法特色**: 使用字串路徑 `"Customer.Address"` 或直接 Include 路徑
- **推薦寫法**: 傳統相容語法

### 語法差異速查

```csharp
// EF Core 3.1.2 (Portal)
.Include(o => o.Customer).ThenInclude(c => c.Address)

// EF6.1.3 (GTS) - 兩種寫法
.Include("Customer.Address")  // 字串路徑
.Include(o => o.Customer.Address)  // 直接指定路徑
```

## 快速決策指南

### 選擇流程圖

```
需要操作資料？
├─ 是 → 使用 Include/ThenInclude（完整實體）
└─ 否 → 只需特定欄位？
    ├─ 是 → 使用 Select 投影（輕量化）
    └─ 否 → 使用 Include/ThenInclude（保持彈性）
```

### 應用場景速查

| 使用情境       | 推薦方法                | 理由         |
| ---------- | ------------------- | ---------- |
| **API 回傳** | Select 投影           | 減少傳輸量，提升效能 |
| **資料更新**   | Include/ThenInclude | 需要完整實體物件   |
| **報表查詢**   | Select 投影           | 通常只需特定欄位   |
| **業務邏輯**   | Include/ThenInclude | 可能需要存取多個屬性 |

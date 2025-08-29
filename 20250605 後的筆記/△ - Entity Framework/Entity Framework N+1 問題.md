---
date: 2025-07-22 23:25
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
## 什麼是 N+1 問題？

**定義：** 執行 1 次主查詢 + N 次額外查詢，總共 N+1 次資料庫查詢的效能陷阱。

## 問題產生的原因

### 範例資料結構

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; }
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

### 產生 N+1 問題的程式碼

```csharp
// ❌ 錯誤寫法
var orders = context.Orders.ToList(); // 1 次查詢

foreach (var order in orders) 
{
    // 每次都會觸發一次資料庫查詢！
    Console.WriteLine($"訂單 {order.Id} - 客戶：{order.Customer.Name}");
    // ↑ 產生 N 次查詢（N = 訂單數量）
}
```

### 實際產生的 SQL

```sql
-- 第 1 次查詢：主查詢
SELECT Id, CustomerId FROM Orders

-- 第 2-N+1 次查詢：每筆訂單的客戶資料
SELECT Id, Name FROM Customers WHERE Id = 1
SELECT Id, Name FROM Customers WHERE Id = 2
SELECT Id, Name FROM Customers WHERE Id = 3
-- ... 以此類推
```

**如果有 100 筆訂單 = 1 + 100 = 101 次查詢！**

## 解決方案

### 方案 1: 使用 Include（預先載入）

```csharp
// ✅ 正確寫法 - 只會產生 1 次查詢
var orders = context.Orders
    .Include(o => o.Customer)  // 預先載入客戶資料
    .ToList();

foreach (var order in orders) 
{
    // 不會觸發額外查詢，資料已經載入
    Console.WriteLine($"訂單 {order.Id} - 客戶：{order.Customer.Name}");
}
```

**產生的 SQL：**

```sql
-- 只有 1 次查詢
SELECT o.Id, o.CustomerId, c.Id, c.Name
FROM Orders o
LEFT JOIN Customers c ON o.CustomerId = c.Id
```

### 方案 2: 使用 Select 投影（推薦）

```csharp
// ✅ Select 投影寫法 - 效能最佳
var orderData = context.Orders
    .Select(o => new {
        OrderId = o.Id,
        CustomerName = o.Customer.Name,
        OrderDate = o.OrderDate
    })
    .ToList();
```

**產生的 SQL：**

```sql
SELECT [o].[Id] AS [OrderId], 
       [o].[OrderDate], 
       [c].[Name] AS [CustomerName]
FROM [Orders] AS [o]
INNER JOIN [Customers] AS [c] ON [o].[CustomerId] = [c].[Id]
```

### 多層關聯的 Select 投影

```csharp
var orderData = context.Orders
    .Select(o => new {
        OrderId = o.Id,
        CustomerName = o.Customer.Name,
        CustomerCity = o.Customer.Address.City,
        CustomerCountry = o.Customer.Address.Country,
        OrderItemCount = o.OrderItems.Count()
    })
    .ToList();
```

**產生的 SQL：**

```sql
SELECT [o].[Id] AS [OrderId],
       [c].[Name] AS [CustomerName],
       [a].[City] AS [CustomerCity],
       [a].[Country] AS [CustomerCountry],
       (
           SELECT COUNT(*)
           FROM [OrderItems] AS [oi]
           WHERE [oi].[OrderId] = [o].[Id]
       ) AS [OrderItemCount]
FROM [Orders] AS [o]
INNER JOIN [Customers] AS [c] ON [o].[CustomerId] = [c].[Id]
LEFT JOIN [Addresses] AS [a] ON [c].[AddressId] = [a].[Id]
```

## Select 投影的優勢

1. **效能優勢**
    
    - 單一查詢：避免多次資料庫往返
    - 減少資料傳輸：只取需要的欄位
    - 記憶體效率：不載入完整的實體物件
    
2. **完全避免 N+1 問題**
    
    - 只產生一次 SQL 查詢
    - 自動產生適當的 JOIN 語句
    - 只傳輸需要的資料
    - 不會觸發懶載入

## 重點提醒

- **N+1 問題 = 1 次主查詢 + N 次懶載入查詢**
- **解決關鍵：預先載入相關資料，避免在迴圈中觸發額外查詢**
- **推薦使用 Select 投影，特別適合 API 和報表場景**
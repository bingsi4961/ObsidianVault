---
date: 2025-07-23 11:55
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
## 基本概念說明

### Include

- **直接關聯載入**：用於載入與當前實體直接關聯的實體
- **第一層關係**：僅能載入與查詢實體有直接關係的實體
- **語法**：`context.Entity.Include(e => e.RelatedEntity)`

### ThenInclude (僅限 EF Core)

- **深層關聯載入**：用於載入間接關聯的實體（關聯的關聯）
- **多層次關係**：能夠在 `Include` 之後繼續深入到下一層關係
- **語法**：`context.Entity.Include(e => e.RelatedEntity).ThenInclude(r => r.DeeperRelatedEntity)`

## 實例說明

假設有以下實體關係：

- `Order` (訂單) 有多個 `OrderItem` (訂單項目)
- 每個 `OrderItem` 關聯一個 `Product` (產品)
- `Order` 關聯一個 `Customer` (客戶)
- `Customer` 關聯一個 `Address` (地址)

## EF6 vs EF Core 的語法差異

### EF6 - 自動推斷中間路徑

#### 單層關聯

```csharp
// 載入訂單及其訂單項目
var orders = context.Orders.Include(o => o.OrderItems).ToList();
```

#### 多層關聯 - Lambda 自動推斷（推薦）

```csharp
// ✅ 正確：只需要這一行，EF6 會自動載入中間的 Customer
var orders = context.Orders
    .Include(o => o.Customer.Address)
    .ToList();

// ❌ 多餘：不需要額外 Include Customer
var orders = context.Orders
    .Include(o => o.Customer)           // 這行是多餘的！
    .Include(o => o.Customer.Address)   
    .ToList();
```

#### 複合關聯載入

```csharp
// 同時載入多個關聯
var orders = context.Orders
    .Include(o => o.OrderItems.Select(oi => oi.Product))  // 載入 OrderItems 和 Product
    .Include(o => o.Customer.Address)
    .ToList();
```

#### EF6 Select 語法的特殊性

```csharp
// EF6 的 Select 在 Include 中不是投影，而是關聯導航語法
.Include(o => o.Items.Select(i => i.Product))
// 會同時載入：
// 1. Items 集合（完整的 Item 物件）
// 2. 每個 Item 對應的 Product（完整的 Product 物件）

// 等價於 EF Core 的：
.Include(o => o.Items)
.ThenInclude(i => i.Product)
```

### EF Core - 條件支援自動推斷

#### 使用 ThenInclude 語法（推薦）

```csharp
// ✅ EF Core 推薦語法
var orders = context.Orders
    .Include(o => o.Customer)
    .ThenInclude(c => c.Address)
    .ToList();

// 載入訂單、訂單項目，以及每個訂單項目對應的產品
var orders = context.Orders
    .Include(o => o.OrderItems)
    .ThenInclude(oi => oi.Product)
    .ToList();
```

#### Navigation Chain 語法（條件支援）

```csharp
// ✅ 條件支援：當路徑全部是 Reference Navigation 時可用
var orders = context.Orders
    .Include(o => o.Customer.Address)  // 只有當 Customer 和 Address 都是單一實體關聯
    .ToList();
```

#### 複合關聯載入

```csharp
// 同時載入多個關聯
var orders = context.Orders
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
    .Include(o => o.Customer)
        .ThenInclude(c => c.Address)
    .ToList();
```

## Navigation 類型說明

### Reference Navigation（參考導航）

**指向單一實體的屬性**（一對一、多對一關係）

```csharp
public class Order
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public Customer Customer { get; set; }  // ← 這是 Reference Navigation
}

public class Customer  
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int AddressId { get; set; }
    public Address Address { get; set; }    // ← 這是 Reference Navigation
}

public class Address
{
    public int Id { get; set; }
    public string Street { get; set; }
    public string City { get; set; }
}
```

### Collection Navigation（集合導航）

**指向多個實體的集合屬性**（一對多、多對多關係）

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Order> Orders { get; set; } // ← 這是 Collection Navigation
}

public class Order
{
    public int Id { get; set; }
    public List<OrderItem> Items { get; set; } // ← 這是 Collection Navigation
}
```

## EF Core Navigation Chain 規則詳解

### 核心原理

**Collection Navigation 是集合型態，沒有具體的屬性名稱，因此不能在 Collection Navigation 後面直接接 Reference Navigation。**

例如：`List<Order>` 是集合型態，EF Core 無法知道要存取集合中哪個元素的屬性。

### ✅ **情況一：整個路徑都是 Reference Navigation**

```csharp
// Order → Customer → Address (全部都是 Reference)
var orders = context.Orders
    .Include(o => o.Customer.Address)  // ✅ 可以使用！
    .ToList();
```

**為什麼可以？** 因為：

- `Order.Customer` 是 Reference Navigation（一個訂單對應一個客戶）
- `Customer.Address` 是 Reference Navigation（一個客戶對應一個地址）

### ✅ **情況二：以單一 Collection Navigation 結束**

```csharp
// Order → Customer → Orders (前面是 Reference，最後是 Collection)
var orders = context.Orders
    .Include(o => o.Customer.Orders)  // ✅ 可以使用！
    .ToList();
```

**為什麼可以？** 因為：

- `Order.Customer` 是 Reference Navigation
- `Customer.Orders` 是 Collection Navigation（但是在最後）

### ❌ **情況三：路徑中間有 Collection Navigation**

```csharp
// Customer → Orders → Items (中間有 Collection)
var customers = context.Customers
    .Include(c => c.Orders.Items)  // ❌ 不能這樣使用！
    .ToList();
```

**為什麼不可以？** 因為：

- `Customer.Orders` 是 Collection Navigation，型態是 `List<Order>`
- `List<Order>` 是集合，沒有 `Items` 屬性
- EF Core 不知道要存取集合中哪個 Order 的 Items

**必須改用 ThenInclude：**

```csharp
var customers = context.Customers
    .Include(c => c.Orders)           // 先載入 Collection
    .ThenInclude(o => o.Items)        // 告訴 EF Core 對每個 Order 載入其 Items
    .ToList();
```

## 查詢結果差異示例

### 僅使用 Include

```csharp
var orders = context.Orders.Include(o => o.OrderItems).ToList();
```

**查詢結果**：

```
Order {
  ID: 1,
  Date: "2023-05-01",
  OrderItems: [
    { ID: 101, Quantity: 2, Price: 50, ProductID: 201 },
    { ID: 102, Quantity: 1, Price: 75, ProductID: 202 }
  ]
}
```

注意：只能看到 OrderItems 中的 ProductID，沒有實際的 Product 物件資料。

### 使用深層關聯載入

```csharp
// EF6
var orders = context.Orders
    .Include(o => o.OrderItems.Select(oi => oi.Product))
    .ToList();

// EF Core
var orders = context.Orders
    .Include(o => o.OrderItems)
    .ThenInclude(oi => oi.Product)
    .ToList();
```

**查詢結果**：

```
Order {
  ID: 1,
  Date: "2023-05-01",
  OrderItems: [
    { 
      ID: 101, 
      Quantity: 2, 
      Price: 50, 
      ProductID: 201,
      Product: { ID: 201, Name: "筆記型電腦", Description: "高效能筆電" }
    },
    { 
      ID: 102, 
      Quantity: 1, 
      Price: 75, 
      ProductID: 202,
      Product: { ID: 202, Name: "滑鼠", Description: "無線滑鼠" }
    }
  ]
}
```

## EF6 自動推斷中間路徑的重要特性

### 自動推斷中間路徑

當您使用 `.Include(o => o.Customer.Address)` 時：

1. **EF6 會自動推斷路徑**：要存取 Address，必須先經過 Customer
2. **自動載入中間實體**：Customer 會自動被載入
3. **產生適當的 JOIN**：SQL 會包含必要的 JOIN 語句

### 實際產生的 SQL

```csharp
var orders = context.Orders
    .Include(o => o.Customer.Address)
    .ToList();
```

會產生類似這樣的 SQL：

```sql
SELECT 
    [o].[Id], [o].[OrderDate], [o].[CustomerId],
    [c].[Id], [c].[Name], [c].[Email],           -- Customer 自動被載入
    [a].[Id], [a].[Street], [a].[City]           -- Address 被載入
FROM [Orders] AS [o]
LEFT JOIN [Customers] AS [c] ON [o].[CustomerId] = [c].[Id]
LEFT JOIN [Addresses] AS [a] ON [c].[AddressId] = [a].[Id]
```

## 驗證載入狀況

### EF6 自動推斷驗證

```csharp
// 測試程式碼
var orders = context.Orders
    .Include(o => o.Customer.Address)
    .ToList();

foreach (var order in orders)
{
    // 這些都不會觸發額外查詢
    Console.WriteLine($"訂單 ID: {order.Id}");
    Console.WriteLine($"客戶名稱: {order.Customer.Name}");      // Customer 已載入
    Console.WriteLine($"客戶城市: {order.Customer.Address.City}"); // Address 已載入
}
```

### EF6 Select 語法驗證

```csharp
var orders = context.Orders
    .Include(o => o.Items.Select(i => i.Product))
    .ToList();

foreach (var order in orders)
{
    Console.WriteLine($"Order {order.Id}:");
    Console.WriteLine($"Items count: {order.Items.Count}");  // Items 確實被載入
    
    foreach (var item in order.Items)  // Items 集合有完整資料
    {
        Console.WriteLine($"  Item: {item.Name}");           // Item 物件完整
        Console.WriteLine($"  Product: {item.Product.Name}"); // Product 也被載入
    }
}
```

## 主要差異總結

|特性|EF6|EF Core|
|---|---|---|
|Lambda 自動推斷|✅ 無條件支援|⚠️ 條件支援|
|ThenInclude 語法|❌ 不支援|✅ 完整支援|
|集合關聯語法|`.Include(o => o.Items.Select(i => i.Product))`|`.Include(o => o.Items).ThenInclude(i => i.Product)`|
|自動中間路徑|✅ 自動載入|⚠️ 有條件限制|

## 最佳實務建議

### EF6

```csharp
// ✅ 推薦：利用自動推斷特性
var orders = context.Orders
    .Include(o => o.Customer.Address)
    .Include(o => o.OrderItems.Select(oi => oi.Product))
    .ToList();
```

### EF Core

```csharp
// ✅ 推薦：統一使用 ThenInclude，避免記憶複雜規則
var orders = context.Orders
    .Include(o => o.Customer)
        .ThenInclude(c => c.Address)
    .Include(o => o.OrderItems)
        .ThenInclude(oi => oi.Product)
    .ToList();
```

## 效能考量

1. **資料完整性**：    
    - 只用 `Include`：只載入了第一層關係
    - 深層載入：同時載入多層關係的詳細資料
    
2. **查詢效率**：    
    - 一次載入所有需要的資料，避免 N+1 查詢問題
    - 但要注意避免載入過多不需要的資料
    
3. **記憶體使用**：   
    - 深層關聯會增加記憶體使用量
    - 應根據實際需求選擇適當的載入深度

## 總結

- **EF6**：支援 Lambda 自動推斷，`.Include(o => o.Customer.Address)` 會自動載入中間實體
- **EF Core**：Lambda 自動推斷有條件限制，建議統一使用 `ThenInclude` 語法
- **團隊開發**：統一使用 ThenInclude 可避免記憶複雜規則，提高程式碼一致性和維護性
- 兩者都能有效解決關聯資料載入問題，選擇適合的語法提升查詢效率
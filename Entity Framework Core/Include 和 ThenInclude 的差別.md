---
date : 2025-07-10 10:14
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
#### 📑 [[歡迎]]
#### 📑 [[歡迎]]

---
## Include

- **直接關聯載入**：用於載入與當前實體直接關聯的實體
- **第一層關係**：僅能載入與查詢實體有直接關係的實體
- **語法**：`context.Entity.Include(e => e.RelatedEntity)`

## ThenInclude

- **深層關聯載入**：用於載入間接關聯的實體（關聯的關聯）
- **多層次關係**：能夠在 `Include` 之後繼續深入到下一層關係
- **語法**：`context.Entity.Include(e => e.RelatedEntity).ThenInclude(r => r.DeeperRelatedEntity)`

## 實例說明

假設有以下實體關係：
- `Order` (訂單) 有多個 `OrderItem` (訂單項目)
- 每個 `OrderItem` 關聯一個 `Product` (產品)

### 使用 Include

```csharp
// 只載入訂單及其訂單項目
var orders = context.Orders.Include(o => o.OrderItems).ToList();
```

### 使用 ThenInclude

```csharp
// 載入訂單、訂單項目，以及每個訂單項目對應的產品
var orders = context.Orders
               .Include(o => o.OrderItems)
               .ThenInclude(oi => oi.Product)
               .ToList();
```

## 資料結構示例

假設我們有這樣的資料庫結構：

- `Order` (訂單)：包含 ID、日期等基本資訊
- `OrderItem` (訂單項目)：包含數量、單價等，與訂單是一對多關係
- `Product` (產品)：包含名稱、描述等，與訂單項目是一對一關係

## 查詢結果差異

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

注意：在這個結果中，我們只能看到 OrderItems 中的 ProductID，但沒有實際的 Product 物件資料。

### 使用 Include 加 ThenInclude
```csharp
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

## 主要差異

1. **資料完整性**：
   - 只用 `Include`：只載入了 Order 和 OrderItems 的資料
   - 加上 `ThenInclude`：同時載入了 Product 的詳細資料

2. **查詢的深度**：
   - `Include`：只深入一層關係
   - `ThenInclude`：深入到第二層關係

3. **實際SQL查詢**：
   - `Include` 會生成一個 JOIN 查詢（Orders JOIN OrderItems）
   - `ThenInclude` 會生成兩個 JOIN 查詢（Orders JOIN OrderItems JOIN Products）

## 實際使用的影響

如果您之後需要使用產品資訊：
- 只用 `Include`：需要額外查詢資料庫獲取 Product 資訊
- 使用 `ThenInclude`：Product 資訊已經載入，不需要再查詢資料庫

這就是為什麼雖然可能返回相同的訂單筆數，但資料的完整性和後續操作的效率會有很大差異。

## 總結

- `Include` 用於載入直接關聯的實體（一層關係）
- `ThenInclude` 用於載入間接關聯的實體（多層關係）
- 兩者結合使用可以實現多層次的關聯資料載入，優化資料查詢效率
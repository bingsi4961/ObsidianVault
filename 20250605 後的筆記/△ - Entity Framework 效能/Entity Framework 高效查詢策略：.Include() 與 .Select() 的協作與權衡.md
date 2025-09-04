---
date: 2025-09-04 15:23
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
## **一、核心觀念：意圖決定工具**

在撰寫 EF 查詢時，首要釐清您的「意圖」。您是想**塑造唯讀資料**，還是要**取得實體來更新**？這決定了您應該使用哪種工具。

- **`.Select()` (投影)**：
    
    - **意圖**：塑造資料 (Data Shaping)。從資料庫中精準地取出您需要的欄位，組合成一個自訂的物件（DTO 或匿名型別）。
        
    - **特性**：高效能、低網路傳輸量、結果為**唯讀且未被追蹤**。
        
- **`.Include()` (預先載入)**：
    
    - **意圖**：預先載入 (Eager Loading)。取得一個**完整的實體物件**，並同時將其關聯的實體也完整載入到記憶體中。
        
    - **特性**：避免 N+1 查詢問題、結果為**可被 `DbContext` 完整追蹤**的實體，適合後續的資料更新操作。        

---

## **二、基本原則：唯讀查詢與單一屬性**

當您只是要查詢資料用來顯示，且關聯的屬性 (select 投影屬性) **不是集合**時，這是最單純的情境。

**規則：直接使用 `.Select()`，完全不需要 `.Include()`。**

EF 會自動分析您的投影需求並產生必要的 SQL `JOIN`。

C#
```csharp
// 【推薦】
var productInfo = await dbContext.Products
    .Select(p => new 
    {
        ProductName = p.Name,
        CategoryName = p.Category.CategoryName // EF 會自動 JOIN Categories 資料表
    })
    .ToListAsync();
```

SQL
```sql
SELECT [p].[Name] AS [ProductName], [c].[CategoryName] 
FROM [Products] AS [p] INNER JOIN [Categories] AS [c] ON [p].[CategoryId] = [c].[CategoryId]
```

**深入解析：為什麼在此情境下，多餘的 `.Include()` 是個壞習慣？**

這正是您提到的關鍵觀念。假設您寫了以下不推薦的程式碼：

C#
```csharp
// 【不推薦】
var productInfoWithRedundantInclude = await dbContext.Products
    .Include(p => p.Category)  // <--- 在 EF6 中，這個指令會被優先執行
    .Select(p => new { ProductName = p.Name, CategoryName = p.Category.CategoryName })
    .ToListAsync();
```

### 執行邏輯分析

EF6 的處理順序更接近字面意思，會按照以下步驟執行：

1. **優先處理 `.Include()`**：
    - EF6 看到 `.Include(p => p.Category)` 時，會將此視為首要任務
    - 目標是產生能夠載入 `Products` 和 `Categories` **所有欄位** 的 SQL 查詢
    - 這樣才能在記憶體中建構出完整的實體物件
    
2. **執行 SQL 查詢**：    
    - 向資料庫發送包含兩個資料表所有欄位的查詢
    - 實際產生的 SQL 類似：
    
```sql
SELECT 
	p.ProductId, p.Name, p.Description, p.Price, p.Stock, p.CreateDate, -- Products 的所有欄位
	c.CategoryId, c.CategoryName, c.Description, c.ImageUrl, c.IsActive -- Categories 的所有欄位
FROM Products p
LEFT JOIN Categories c ON p.CategoryId = c.CategoryId
```
    
3. **在記憶體中執行 `.Select()`**：
    - 當 SQL 查詢返回龐大的資料集後
    - EF6 的 runtime 才在應用程式記憶體中執行最後的投影
    - 從已載入的完整實體中挑出 `ProductName` 和 `CategoryName`

### 效能問題詳解

**`.Include(p => p.Category)` 造成的浪費**：

- 這個指令要求載入 `Category` 實體的**所有欄位**
- 可能包含：`CategoryId`、`CategoryName`、`Description`、`ImageUrl`、`IsActive`、`CreateDate`、`UpdateDate` 等眾多欄位

**資源浪費流程**：

1. 資料庫查詢並回傳 `Category` 的**所有欄位資料**
2. 大量不需要的資料透過網路傳輸到應用程式
3. EF6 在記憶體中建構完整的 `Category` 實體物件
4. `.Select()` 最後才說：「我只需要 `CategoryName` 一個欄位」
5. 其他欄位（`Description`、`ImageUrl` 等）完全被浪費

### 正確的做法

移除不必要的 `.Include()`，直接使用 `.Select()`：

C#
```csharp
var productInfo = await dbContext.Products
    .Select(p => new { 
        ProductName = p.Name, 
        CategoryName = p.Category.CategoryName 
    })
    .ToListAsync();
```

這樣 EF6 會產生最佳化的 SQL：

SQL
```sql
SELECT 
    p.Name as ProductName,
    c.CategoryName 
FROM Products p
LEFT JOIN Categories c ON p.CategoryId = c.CategoryId
```

### 重點提醒    

**結論：** 雖然新版的 EF Core 優化器通常足夠聰明，可能會忽略多餘的 `.Include()`，但依賴這個行為是個壞習慣。從根本上說，**不必要的 `.Include()` 會導致資料庫執行更繁重的查詢、傳輸更多的網路流量，造成不必要的效能開銷**。反之，直接使用 `.Select()` 投影，EF 產生的 SQL 就只會精準地 `SELECT` 你需要的欄位，效能最佳。

---

## **三、進階情境：基本原則的例外**

##### **情境 1：當投影的屬性是「集合型態」**

**規則：必須先用 `.Include()` 預先載入集合，再於 `.Select()` 中進行處理。**

##### **情境 2：當查詢的目的是為了「更新資料」**

**規則：絕不能使用 `.Select()` 投影，必須查詢回完整的實體，並使用 `.Include()` 載入需要一併處理的關聯實體。**

---

## **四、`.Include()` 的進階用法：過濾載入 (Filtered Include)**

(此處內容與前一版相同，為保持筆記完整性而保留)

**規則：`.Include()` 不能指定載入特定「欄位」，但從 EF Core 5.0 開始，可以過濾要載入的「資料列」。**

---

## **五、最終決策樹 (Summary)**

|您的意圖是什麼？|關聯屬性是集合嗎？|最佳做法|
|---|---|---|
|**唯讀查詢 (Read-Only)**|否 (單一物件)|直接 `.Select()` 投影，**不要**用 `.Include()` 以避免載入不必要的大資料。|
|**唯讀查詢 (Read-Only)**|是 (集合 `ICollection<T>`)|**先 `.Include()`** 載入集合，**再 `.Select()`** 投影以避免 N+1 問題。|
|**更新資料 (Update)**|(不論是否為集合)|**不要**用 `.Select()`，直接查詢實體，並用 `.Include()` 載入需要的關聯資料。|

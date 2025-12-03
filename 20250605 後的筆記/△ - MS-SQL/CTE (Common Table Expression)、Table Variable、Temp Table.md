---
date: 2025-12-03 22:53
aliases:
tags:
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
## 一、CTE (Common Table Expression，通用資料表運算式)

### 基本概念

- **本質**：邏輯查詢，不是實體資料
- **類比**：像 C# 中的 `IQueryable` 或 LINQ Query，定義邏輯但尚未執行

### 語法結構

```sql
;WITH CteName AS (
    SELECT ID, Name FROM Users WHERE Age > 30
)
SELECT * FROM CteName;
```

### 重要特性

1. **為什麼前面要加分號？**    
    - SQL Server 要求前一個語句必須以分號結尾
    - 防禦性寫法：在 WITH 前加分號，避免前一行忘記寫分號
    
2. **有效範圍**    
    - 僅在定義後的「第一個 SQL 語句」中有效
    - 語句結束後立即消失

3. **執行特性**    
    - 每次引用都會重新執行查詢
    - 不會儲存中間結果
    - 多次引用 = 多次執行（效能殺手）

## 二、三者比較表

|特性|CTE|Table Variable (@Table)|Temp Table (#Table)|
|---|---|---|---|
|**C# 類比**|`var` (LINQ Query/IQueryable)|`List<T>` (記憶體變數)|`DataTable` / 實體 Table|
|**儲存位置**|無（邏輯定義）|記憶體（量大時寫入 tempdb）|tempdb 資料庫|
|**有效範圍**|當下那句 SQL|當下 Batch|當下 Session|
|**統計資訊**|使用底層 Table|無（預設估計為 1 筆）⚠️|有（自動維護）|
|**索引支援**|不可建索引|僅能在宣告時定義 PK/Unique|可隨時 CREATE INDEX|
|**適合資料量**|小量、一次性|< 100 筆|大量資料|

## 三、關鍵問題解答

### Q1：CTE 只能使用一次？每次被引用都重新執行？

**答案：是的**

- CTE 是邏輯定義，不是資料儲存
- 多次 JOIN 同一個 CTE = 多次執行內部查詢
- 大數據量時應改用 Temp Table

### Q2：Table Variable 預設只有 1 筆資料？

**答案：SQL Server 的「特性」（坑）**

- Query Optimizer 會假設 Table Variable 只有 1 筆
- 導致選擇不當的執行計畫（如 Nested Loop Join）
- SQL Server 2019 後稍有改善，但大量資料仍應避免使用

## 四、實務範例

### 1. CTE 範例

```sql
;WITH OrderStats AS (
    SELECT 
        CustomerId, 
        SUM(TotalAmount) AS TotalSpent
    FROM Orders
    WHERE OrderDate >= '2024-01-01'
    GROUP BY CustomerId
)
SELECT * FROM OrderStats
WHERE TotalSpent > 10000;
-- OrderStats 在這裡消失
```

### 2. Table Variable 範例

```sql
DECLARE @VipList TABLE (
    CustomerId INT PRIMARY KEY,
    CustomerName NVARCHAR(50)
);

INSERT INTO @VipList (CustomerId, CustomerName)
VALUES (101, 'ASUS User'), (102, 'Gemini User');

SELECT o.OrderId, o.TotalAmount, v.CustomerName
FROM Orders o
JOIN @VipList v ON o.CustomerId = v.CustomerId;
```

### 3. Temp Table 範例

```sql
-- 建立並填入資料
SELECT 
    CustomerId, 
    SUM(TotalAmount) AS TotalSpent
INTO #CustomerStats
FROM Orders
WHERE OrderDate >= '2023-01-01'
GROUP BY CustomerId;

-- 建立索引優化
CREATE CLUSTERED INDEX IX_Temp_CustId ON #CustomerStats(CustomerId);

-- 進行複雜運算
UPDATE #CustomerStats 
SET TotalSpent = TotalSpent * 0.9 
WHERE TotalSpent > 100000;

-- 查詢結果
SELECT * FROM #CustomerStats 
WHERE TotalSpent > 10000;

-- 清理
DROP TABLE #CustomerStats;
```

## 五、開發場景建議

### 何時使用 CTE？

- 提高程式碼可讀性（拆解複雜邏輯）
- 遞迴查詢（組織架構、BOM 表）
- 分頁查詢（配合 ROW_NUMBER()）

### 何時使用 Table Variable？

- 暫存小量資料（< 100 筆）
- 前端傳來的 ID 清單
- Table-Valued Function 回傳值

### 何時使用 Temp Table？

- 資料量大（> 1000 筆）
- 需要多次 JOIN 或複雜運算
- 需要建立索引優化效能
- 處理複雜報表或 ETL 流程

## 六、效能優化建議

1. **避免 CTE 重複引用**
    
    - 大量資料時改用 Temp Table
2. **謹慎使用 Table Variable**
    
    - 只用於極少量資料
    - 大量資料會導致執行計畫錯誤
3. **善用 Temp Table**
    
    - 建立適當索引
    - 記得手動 DROP（雖然會自動清理）
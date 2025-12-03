---
date : 2025-12-03 23:27
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
## 重要觀念

- **NULL 擴散性 (Null Propagation)**：任何東西跟 NULL 做運算，結果通常都是 NULL
    - `1 + NULL = NULL`
    - `'ABC' + NULL = NULL`
    - `TRIM(NULL) = NULL`

## 1. ISNULL() 函式

> 💡 **MySQL 稱為 IFNULL()，但在 SQL Server 是 ISNULL()**

### 功能

如果原值是 NULL，就換成備用值（最常用的「代打」函數）

### 語法

```sql
ISNULL(檢查欄位, 替代值)
```

### C# 對照

```csharp
Price ?? 0  // Null-coalescing operator
```

### 範例

```sql
-- 如果 Phone 是 NULL，就顯示 '無電話'
SELECT Name, ISNULL(Phone, '無電話') 
FROM Customers;
```

## 2. COALESCE() 函式

> 唸法：Ko-a-les

### 功能

ISNULL 的進化版，可接受多個參數，回傳第一個非 NULL 的值

### 語法

```sql
COALESCE(值1, 值2, 值3, ...)
```

### C# 對照

```csharp
var result = val1 ?? val2 ?? val3;
```

### 實務範例

```sql
-- 優先順序：手機 -> 家裡電話 -> 公司電話 -> '無'
SELECT 
    Name,
    COALESCE(Mobile, HomePhone, OfficePhone, '無') AS ContactNumber
FROM Users;
```

### ISNULL vs COALESCE 比較

|特性|ISNULL|COALESCE|
|---|---|---|
|語法來源|T-SQL 專用|標準 ANSI SQL|
|參數數量|只能 2 個|可多個|
|效能|極微幅較快|稍慢一點|
|回傳型別|強制比照第一個參數|以優先級最高的參數為主|
|建議用途|簡單判斷|多欄位判斷|

## 3. NULLIF() 函式

### 功能

如果兩參數相等，回傳 NULL；否則回傳第一個參數

### 語法

```sql
NULLIF(參數1, 參數2)
```

### C# 對照

```csharp
(var1 == var2) ? null : var1
```

### 最強實務場景：避免除以零錯誤

```sql
DECLARE @Total INT = 1000;
DECLARE @Count INT = 0;

-- 這行會報錯：Divide by zero error encountered.
-- SELECT @Total / @Count; 

-- 使用 NULLIF 自救：
SELECT @Total / NULLIF(@Count, 0) AS Average; 
-- NULLIF(0, 0) 會變成 NULL，1000 / NULL = NULL (不報錯)
```

## 4. TRIM 與 NULL

### SQL Server 版本差異

- **SQL Server 2017 以前**（GTS .NET 4.8）
    - 沒有 TRIM() 函式
    - 必須使用：`LTRIM(RTRIM(ColumnName))`
- **SQL Server 2017 以後**（Portal .NET Core 3.1）
    - 直接支援：`TRIM(ColumnName)`

### TRIM(LTRIM(NULL)) 解析

1. 最裡面的 `NULL` 是空值
2. `LTRIM(NULL)` → 結果還是 `NULL`
3. `TRIM(NULL)` → 結果依然是 `NULL`

### 正確處理方式

```sql
-- 如果是 NULL，先轉成空字串，再去空白
SELECT TRIM(ISNULL(MyColumn, ''));
```

## 快速記憶表

|函式|功能簡述|C# 類比|記憶口訣|
|---|---|---|---|
|`ISNULL(A, B)`|A 是空就給 B|`A ?? B`|A 掛了 B 上|
|`COALESCE(A, B, C)`|給第一個有值的|`A ?? B ?? C`|撿剩的|
|`NULLIF(A, B)`|A 等於 B 就給 NULL|`(A==B) ? null : A`|除以零救星|
|`TRIM(NULL)`|結果還是 NULL|`null.Trim()` ⚠️|空就是空|

> ⚠️ **注意**：在 C# 中 `null.Trim()` 會報 NullReferenceException，但 SQL 會回傳 NULL
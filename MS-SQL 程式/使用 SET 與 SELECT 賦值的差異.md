---
title: 使用 SET 與 SELECT 賦值的差異
tags: [MS-SQL 程式]

---

1. SET 的特點：
```sql
-- SET 一次只能給一個變數賦值
SET @Variable = 100

-- SET 的語法更清晰直觀
SET @CurrentDate = GETDATE()

-- SET 在執行計劃中通常更有效率
SET @Count = (SELECT COUNT(*) FROM Table)
```

2. SELECT 的特點：
```sql
-- SELECT 可以同時給多個變數賦值
SELECT 
    @FirstVar = Column1,
    @SecondVar = Column2
FROM Table
WHERE ID = 1

-- SELECT 可以直接從查詢結果賦值
SELECT @TotalAmount = SUM(Amount) 
FROM Orders
WHERE OrderDate = @Date
```

使用建議：

1. 使用 SET 的情況：
- 給單個變數賦值
- 賦值邏輯簡單明確
- 不需要從查詢結果中獲取值
```sql
SET @StartDate = DATEADD(day, -7, GETDATE())
SET @MaxValue = 100
```

2. 使用 SELECT 的情況：
- 需要同時給多個變數賦值
- 需要從查詢結果中獲取值
- 有條件判斷的賦值
```sql
-- 同時賦值多個變數
SELECT 
    @FirstName = FirstName,
    @LastName = LastName,
    @Email = Email
FROM Users 
WHERE UserID = @UserID

-- 條件判斷賦值
SELECT @Status = 
    CASE 
        WHEN Amount > 1000 THEN 'High'
        WHEN Amount > 500 THEN 'Medium'
        ELSE 'Low'
    END
```

3. 要注意的問題：
```sql
-- 這種寫法可能產生意外結果
SELECT @Var = Column1 FROM Table
-- 如果 Table 有多行，@Var 會得到最後一行的值

-- 更安全的寫法
SELECT TOP 1 @Var = Column1 FROM Table
```
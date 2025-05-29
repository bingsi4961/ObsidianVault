---
title: CAST 與 CONVERT 的差異
tags: [MS-SQL 程式]

---

CONVERT 和 CAST 在 SQL Server 中都是用來做資料型別轉換，但有以下差異：

1. 語法差異：
```sql
-- CAST 語法
CAST(運算式 AS 資料型別)

-- CONVERT 語法
CONVERT(資料型別, 運算式 [, 格式碼])
```

2. 主要差異：
- CONVERT 可以指定格式碼（style parameter），特別適合日期和數值格式的轉換
- CAST 語法較簡單，符合 SQL 標準，易於跨資料庫使用
- CONVERT 是 SQL Server 特有的函數

3. 實際應用範例：
```sql
-- 日期轉換
-- CAST
SELECT CAST('2024-01-01' AS DATE)

-- CONVERT （可以指定格式）
SELECT CONVERT(DATE, '2024-01-01', 120)  -- 120 是日期格式代碼
SELECT CONVERT(VARCHAR, GETDATE(), 112)  -- 112 會輸出 YYYYMMDD 格式

-- 數值轉換
-- CAST
SELECT CAST(123.45 AS INT)         -- 結果: 123

-- CONVERT
SELECT CONVERT(INT, 123.45)        -- 結果: 123
```

4. 使用建議：
- 如果需要特定格式（特別是日期），用 CONVERT
- 如果是簡單的型別轉換，用 CAST
- 如果考慮跨資料庫相容性，用 CAST
- 性能上兩者基本相同
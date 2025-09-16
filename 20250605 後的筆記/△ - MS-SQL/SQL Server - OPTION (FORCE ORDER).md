---
date: 2025-09-16 14:40
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

`OPTION (FORCE ORDER)` 是 SQL Server 的查詢提示 (Query Hint)，用來強制 SQL Server 按照您在 `FROM` 和 `JOIN` 子句中寫的順序來執行表的連接。

## 基本概念

### **沒有 FORCE ORDER 時（預設行為）**

```sql
SELECT * FROM A 
JOIN B ON A.id = B.id 
JOIN C ON B.id = C.id
-- SQL Server 會自動決定最佳的 JOIN 順序，可能是：
-- A → B → C 或 B → A → C 或 C → B → A 等等
```

### **使用 FORCE ORDER 時**

```sql
SELECT * FROM A 
JOIN B ON A.id = B.id 
JOIN C ON B.id = C.id
OPTION (FORCE ORDER);
-- 強制按照 A → B → C 的順序執行
```

## 實際應用場景

### **您的查詢案例**

```sql
SELECT * FROM SOAExpandPartsList AS list          -- 第1個表
    INNER JOIN SOA_CFG_BOM_HEADER AS header        -- 第2個表  
        ON list.WORK_NO = header.WORK_NO
    INNER JOIN SOAExpandPartsResult AS result      -- 第3個表
        ON header.TX_ID = result.TX_ID
    INNER JOIN SOA_BOM_SMALL_EXPL_TEMP_V AS tempv  -- 第4個表（問題表）
        ON result.TX_ID = tempv.TX_ID
WHERE list.FileName = '...' 
    AND list.WORK_NO LIKE 'e9b-%'
OPTION (FORCE ORDER);
```

**執行順序變成：**

1. 先從 `SOAExpandPartsList` 開始過濾
2. 再 JOIN `SOA_CFG_BOM_HEADER`
3. 接著 JOIN `SOAExpandPartsResult`
4. 最後才 JOIN `SOA_BOM_SMALL_EXPL_TEMP_V`

## 為什麼可能有效？

### **原本的問題**

SQL Server 的查詢優化器可能選擇了錯誤的執行計畫，例如：

```sql
-- 可能的錯誤執行順序：
-- 1. 先掃描大表 SOA_BOM_SMALL_EXPL_TEMP_V
-- 2. 再與其他表 JOIN
-- 3. 最後才套用 WHERE 條件過濾
-- → 導致處理大量不必要的資料
```

### **FORCE ORDER 的效果**

```sql
-- 強制的執行順序：
-- 1. 先用 WHERE 條件過濾 SOAExpandPartsList (資料量小)
-- 2. 逐步與其他表 JOIN
-- 3. 最後才碰 SOA_BOM_SMALL_EXPL_TEMP_V (此時資料量已經很小)
-- → 大幅減少需要處理的資料量
```

## 使用注意事項

### **優點**

- 可以解決查詢優化器選錯執行計畫的問題
- 對於複雜查詢，有時能大幅改善效能

### **缺點**

- 會失去查詢優化器的自動優化能力
- 如果資料分佈改變，可能反而變慢
- 不是所有情況都適用

### **建議使用時機**

```sql
-- 適合使用的情況：
-- 1. 確定知道最佳的 JOIN 順序
-- 2. 查詢優化器持續選錯執行計畫
-- 3. 有明確的過濾條件應該先執行

-- 不建議使用的情況：
-- 1. 資料量經常變動
-- 2. 不確定最佳執行順序
-- 3. 查詢已經夠快了
```

## 其他相關的 Query Hint

```sql
-- 其他常用的查詢提示
OPTION (FORCE ORDER)           -- 強制 JOIN 順序
OPTION (HASH JOIN)             -- 強制使用 Hash Join
OPTION (MERGE JOIN)            -- 強制使用 Merge Join
OPTION (LOOP JOIN)             -- 強制使用 Nested Loop Join
OPTION (MAXDOP 1)              -- 限制平行處理程度
OPTION (RECOMPILE)             -- 每次執行都重新編譯執行計畫
```

在您的案例中，由於 `SOA_BOM_SMALL_EXPL_TEMP_V` 是效能瓶頸，使用 `FORCE ORDER` 確保它在最後處理，應該會有不錯的效果。
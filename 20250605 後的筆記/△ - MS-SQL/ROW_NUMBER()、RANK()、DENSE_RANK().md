---
date: 2025-11-12 10:49
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
## 一、ROW_NUMBER() 是什麼？

**ROW_NUMBER()** 是一個**視窗函數 (Window Function)**，用來為查詢結果集中的每一行指派一個唯一且連續的整數（行號），例如 1, 2, 3, 4...

### 您的 SQL 範例解析

```sql
SELECT *,
       ROW_NUMBER() OVER (PARTITION BY NpilogFileId, FileName 
                          ORDER BY ModifyDate DESC) AS rn
FROM NpiLogFileInfo
```

### OVER 子句說明

#### 1. PARTITION BY NpilogFileId, FileName

- **分組功能**：將 `NpilogFileId` 和 `FileName` 兩欄完全相同的資料列視為一個獨立的群組（分割區）
- 每個群組內會重新從 1 開始編號

#### 2. ORDER BY ModifyDate DESC

- **排序功能**：在每個群組內部，依照 `ModifyDate`（修改日期）從最新到最舊排序
- 最新的資料會被編號為 1

### 實際範例

**原始資料：**

|NpilogFileId|FileName|ModifyDate|...其他欄位...|
|---|---|---|---|
|1|'A.txt'|2025-11-11|...|
|1|'A.txt'|2025-11-10|...|
|1|'B.txt'|2025-11-09|...|
|2|'C.txt'|2025-11-05|...|
|2|'C.txt'|2025-11-01|...|

**查詢結果：**

|NpilogFileId|FileName|ModifyDate|...其他欄位...|rn|
|---|---|---|---|---|
|1|'A.txt'|2025-11-11|...|1|
|1|'A.txt'|2025-11-10|...|2|
|1|'B.txt'|2025-11-09|...|1|
|2|'C.txt'|2025-11-05|...|1|
|2|'C.txt'|2025-11-01|...|2|

### 最常見的用途：取得每個群組的最新資料

```sql
/* 使用 CTE (Common Table Expression) */
WITH RankedLogs AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY NpilogFileId, FileName 
                              ORDER BY ModifyDate DESC) AS rn
    FROM NpiLogFileInfo
)
/* 只選取最新的資料 (rn = 1) */
SELECT *
FROM RankedLogs
WHERE rn = 1;
```

---

## 二、RANK() vs DENSE_RANK() vs ROW_NUMBER()

這三個視窗函數在處理**相同值（並列）**時的行為完全不同：

### 核心差異

- **ROW_NUMBER()**：不管值是否相同，就是給每一行一個唯一的連續編號
- **RANK()**：相同值給相同排名，但會**跳號**（就像奧運頒獎）
- **DENSE_RANK()**：相同值給相同排名，但**不跳號**（就像學校排名）

### 範例說明

**原始資料：員工銷售業績**

|Name|Sales|
|---|---|
|Amy|100|
|Bob|90|
|Cathy|80|
|David|80|
|Elva|70|
|Frank|70|
|Gaby|60|

**SQL 查詢：**

```sql
SELECT
    Name,
    Sales,
    ROW_NUMBER() OVER (ORDER BY Sales DESC) AS RowNum,
    RANK()       OVER (ORDER BY Sales DESC) AS Rank,
    DENSE_RANK() OVER (ORDER BY Sales DESC) AS DenseRank
FROM SalesReport;
```

**查詢結果比較：**

|Name|Sales|ROW_NUMBER()|RANK()|DENSE_RANK()|
|---|---|---|---|---|
|Amy|100|1|1|1|
|Bob|90|2|2|2|
|Cathy|80|3|3|3|
|David|80|4|**3**|**3**|
|Elva|70|5|**5**|**4**|
|Frank|70|6|**5**|**4**|
|Gaby|60|7|7|5|

### 詳細行為分析

#### 1. ROW_NUMBER()（流水號）

- 單純指派唯一連續編號（1, 2, 3, 4, 5...）
- Cathy 和 David 雖然同分 80，但還是給了不同編號（3 和 4）

#### 2. RANK()（排名，會跳號）

- Cathy 和 David 同為 80 分 → 並列第 3 名（RANK = 3）
- 因為第 3 名有兩人，佔掉第 3 和第 4 的位置
- 下一位 Elva（70分）直接跳到第 5 名（**跳過了 4**）
- **口訣**：就像奧運頒獎，兩人並列銀牌（第 2），下一位就是第 4 名（銅牌從缺）

#### 3. DENSE_RANK()（密集排名，不跳號）

- Cathy 和 David 同為 80 分 → 並列第 3 名（DENSE_RANK = 3）
- 只在乎「值的群組」（100 是第 1 組，90 是第 2 組，80 是第 3 組...）
- 下一位 Elva（70分）緊接著是第 4 名（**不跳號**）
- **口訣**：就像學校排名，兩人並列第 2 名，下一位還是第 3 名

---

## 三、使用時機

### ROW_NUMBER()

- **目的**：取得唯一的行號、分頁、刪除重複資料
- **情境**：找出「每個群組的最新一筆」（像您的 SQL 範例）

### RANK()

- **目的**：傳統的「名次」排名，反映並列所佔用的名額
- **情境**：找出「業績前 10 名的員工」（如果第 10 名有 3 人並列，他們都會被選中）

### DENSE_RANK()

- **目的**：連續的排名，找出「第 N 高的值」
- **情境**：找出「業績第三高的員工」（在範例中，Cathy 和 David 的分數 80 是第三高）

---

## 四、補充說明

### CTE (Common Table Expression) 用法

使用 CTE 可以讓 SQL 更易讀，特別是在需要先計算視窗函數，再進行篩選的情境。

### 在您的工作環境中

- Portal 系統（.Net Core 3.1）和 GTS 系統（.Net Framework 4.8）都可以使用這些 SQL 視窗函數
- 如果使用 Entity Framework，可以透過 LINQ 的 `FromSqlRaw` 或 `FromSqlInterpolated` 來執行包含視窗函數的原生 SQL
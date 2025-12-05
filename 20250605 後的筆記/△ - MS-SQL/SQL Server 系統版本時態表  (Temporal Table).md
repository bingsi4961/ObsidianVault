---
date: 2025-12-05 09:26
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
## 一、什麼是系統版本時態表？

系統版本時態表是 SQL Server 2016 之後推出的功能，用來**自動記錄資料的變更歷史**。

- 當你修改或刪除資料時，舊的資料會自動被搬移到「歷史資料表」中
- 可以隨時查詢「過去某個時間點」的資料狀態
- 適合用來做 Audit Log (稽核紀錄)

## 二、啟用時態表的 SQL 語法

### 第一步：修改資料表結構（加入時間戳記欄位）

```sql
ALTER TABLE [dbo].[SW_CheckLink] -- (改成要記錄的 Table 名稱)
ADD
  -- 1. 定義「開始時間」欄位
  FromTime DATETIME2(2) GENERATED ALWAYS AS ROW START -- HIDDEN (加入 HIDDEN，需指定欄位名稱(FromTime) 才會顯示)
        CONSTRAINT CheckLink_FromTime DEFAULT DATEADD(SECOND, -1, SYSUTCDATETIME())
  
  -- 2. 定義「結束時間」欄位
, ToTime DATETIME2(2) GENERATED ALWAYS AS ROW END   
        CONSTRAINT CheckLink_ToTime DEFAULT '9999.12.31 23:59:59.99'
  
  -- 3. 定義這兩個欄位為「系統時間區間」
, PERIOD FOR SYSTEM_TIME (FromTime, ToTime);
GO
```

### 第二步：啟用版本控制

```sql
ALTER TABLE [dbo].[SW_CheckLink] -- (改成要記錄的 Table 名稱)
    SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.SW_CheckLinkHistory));  -- (改成要記錄的「Table 名稱 + History」)
GO
```

## 三、關鍵概念解說

### 1. 欄位說明

| 項目                     | 說明             |
| ---------------------- | -------------- |
| **FromTime**           | 欄位名稱（資料的生效時間）  |
| **ToTime**             | 欄位名稱（資料的失效時間）  |
| **CheckLink_FromTime** | 約束條件名稱（不是欄位名稱） |
| **CheckLink_ToTime**   | 約束條件名稱（不是欄位名稱） |

### 2. 重要關鍵字

- **DATETIME2(2)**: 精確度到小數點後兩位
- **GENERATED ALWAYS AS ROW START**: 系統自動維護的「資料生效時間」
- **GENERATED ALWAYS AS ROW END**: 系統自動維護的「資料失效時間」
- **HIDDEN**: 加上此關鍵字後，SELECT * 時不會顯示這兩個時間欄位（建議使用）
- **SYSUTCDATETIME()**: 取得 UTC 時間（時態表強制使用 UTC）
- **PERIOD FOR SYSTEM_TIME**: 告訴 SQL Server 這兩個欄位用來計算資料的有效存活期間

### 3. 預設值說明

**FromTime 的預設值：** `DATEADD(SECOND, -1, SYSUTCDATETIME())`

- 意義：現在的 UTC 時間減 1 秒
- 原因：確保既有資料在啟用版本控制時，狀態已經是「已生效」
- 只在 ALTER TABLE 當下影響既有資料，之後新增資料會自動填入當下時間

**ToTime 的預設值：** `'9999.12.31 23:59:59.99'`

- 意義：「直到永遠」（無限大）
- 原因：現行有效的資料還沒有「結束時間」
- 只要看到 9999 年，就代表這是「目前最新、有效」的資料

### 4. ROW START 與 ROW END 的意義

不是指 SQL 指令執行的「開始」與「結束」，而是指**資料的生命週期**：

- **ROW START（生效時間）**: 這筆資料從什麼時候開始有效的
- **ROW END（失效時間）**: 這筆資料到什麼時候變成舊的/無效的

## 四、實際運作流程

### 新增 (INSERT)

- 資料進入主表 `SW_CheckLink`
- FromTime = 現在 (UTC)
- ToTime = 9999/12/31
- 歷史表為空

### 修改 (UPDATE)

1. 舊資料複製到歷史表 `SW_CheckLinkHistory`
    - FromTime = 原本時間
    - ToTime = 修改當下的時間

2. 主表資料被更新
    - FromTime = 修改當下的時間
    - ToTime = 9999/12/31

### 刪除 (DELETE)

- 資料移入歷史表，ToTime 標記為刪除時間
- 主表該筆資料消失

## 五、查詢歷史資料

### 1. 核心觀念

- 時態表內部儲存 **強制使用 UTC 時間**
- 台灣是 UTC+8

### 2. 查詢所有資料（現行 + 歷史）

```sql
SELECT     
    -- UTC 轉台北時間
    CAST(FromTime AT TIME ZONE 'UTC' AT TIME ZONE 'Taipei Standard Time' AS DATETIME) 
    AS FromTime_TW
    
    -- UTC 轉台北時間（包含溢位保護）
    ,CASE 
        WHEN ToTime < '9999-01-01' 
        THEN CAST(ToTime AT TIME ZONE 'UTC' AT TIME ZONE 'Taipei Standard Time' AS DATETIME)
        ELSE ToTime 
    END AS ToTime_TW

	,*
FROM [dbo].[SW_CheckLink] FOR SYSTEM_TIME ALL
WHERE Id = 286 
ORDER BY FromTime DESC;
```

**重點：**

- `FOR SYSTEM_TIME ALL` 會自動 UNION 主表與歷史表
- 必須處理 9999 年的溢位問題，避免時區轉換報錯

### 3. 查詢 **1**小時前 的資料狀態

```sql
-- 宣告變數：1 小時前的 UTC 時間
DECLARE @OneHourAgo DATETIME2 = DATEADD(hour, -1, SYSUTCDATETIME());

SELECT     
    -- UTC 轉台北時間
    CAST(FromTime AT TIME ZONE 'UTC' AT TIME ZONE 'Taipei Standard Time' AS DATETIME) 
    AS FromTime_TW
    
    -- UTC 轉台北時間（包含溢位保護）
    ,CASE 
        WHEN ToTime < '9999-01-01' 
        THEN CAST(ToTime AT TIME ZONE 'UTC' AT TIME ZONE 'Taipei Standard Time' AS DATETIME)
        ELSE ToTime 
    END AS ToTime_TW
    
    ,*
FROM [dbo].[SW_CheckLink] FOR SYSTEM_TIME AS OF @OneHourAgo
WHERE Id = 286;
```

### 4. 關鍵差異說明

| 語法                           | 用途            | 結果                          |
| ---------------------------- | ------------- | --------------------------- |
| `FOR SYSTEM_TIME ALL`        | 查詢所有資料（現行+歷史） | 會列出所有版本，按時間排序               |
| `FOR SYSTEM_TIME AS OF @時間點` | 查詢特定時間點的狀態    | ==只會顯示 **1 筆**（該時間點有效的那筆）== |

## 六、注意事項

1. **HIDDEN 關鍵字的重要性**    
    - <mark style="background: #FFB86CA6;">建議加上 HIDDEN，避免既有程式因多了兩個欄位而出錯</mark>
    - 需要查詢時必須明確指定欄位名稱

2. **時區轉換的溢位保護**    
    - 9999-12-31 不能直接加 8 小時（會超出範圍）
    - 必須用 CASE WHEN 判斷是否為歷史資料

3. **約束條件命名**    
    - 給約束條件取名，日後修改或刪除時較方便
    - 若不取名，系統會自動產生難記的亂數名稱

4. **強制使用 UTC**    
    - 無法改變，這是時態表的設計規則
    - 避免日光節約時間或時區變更造成時間錯亂
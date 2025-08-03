---
date: 2025-08-03 23:13
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

```csharp
// List<string>
var workNos = valid90PartItems.Select(item => item.WorkNo).ToList();

var result = _taskDb.SoaCfgBomHeader
    .Where(x => workNos.Contains(x.WorkNo.Trim())) // 直接使用 x.WorkNo
    .Join(
        _taskDb.SoaexpandPartsResult,
        header => header.TxId,
        parts => parts.TxId,
        (header, parts) => new { BomHeader = header, ExpandParts = parts }
    )
    .Where(combined => combined.ExpandParts.IsSuccess == true)
    .Join(
        _taskDb.SoaBomSmallExplTempV,
        combined => combined.ExpandParts.TxId,
        temp => temp.TxId,
        (combined, temp) => new
        {
            BomHeader = combined.BomHeader,
            ExpandParts = combined.ExpandParts,
            TempV = temp
        }
    )
    .Select(combined2 => new Expand90BomSOAResult
    {
        WORK_NO = combined2.BomHeader.WorkNo,
        TX_ID = combined2.TempV.TxId,
        TOP_ITEM = combined2.TempV.TopItem,
        TOP_ITEM_DESC = combined2.TempV.TopItemDesc,
        PARENT_ITEM = combined2.TempV.ParentItem,
        COMPONENT_ITEM = combined2.TempV.ComponentItem,
        COMPONENT_ITEM_DESC = combined2.TempV.ComponentItemDesc,
        COMPONENT_QUANTITY = combined2.TempV.ComponentQuantity,
        IS_SUB = combined2.TempV.IsSub
    })
    .ToList();
```

根據您的資料表關係：

- `SoaCfgBomHeader` 一對一 `SoaexpandPartsResult`
- `SoaexpandPartsResult` 一對多 `SoaBomSmallExplTempV`

## 資料筆數說明

假設有以下資料：

**SoaCfgBomHeader (1筆)**

```
WORK_NO: "W001", TX_ID: "T001"
```

**SoaexpandPartsResult (1筆)**

```
TX_ID: "T001", IsSuccess: true
```

**SoaBomSmallExplTempV (3筆)**

```
TX_ID: "T001", COMPONENT_ITEM: "C001"
TX_ID: "T001", COMPONENT_ITEM: "C002" 
TX_ID: "T001", COMPONENT_ITEM: "C003"
```

## 最終結果會是 3 筆

csharp

```csharp
// result 會包含 3 筆 Expand90BomSOAResult
// 每一筆都有相同的 WORK_NO 和 TX_ID
// 但 COMPONENT_ITEM 等欄位會不同

result[0]: WORK_NO="W001", TX_ID="T001", COMPONENT_ITEM="C001"
result[1]: WORK_NO="W001", TX_ID="T001", COMPONENT_ITEM="C002"
result[2]: WORK_NO="W001", TX_ID="T001", COMPONENT_ITEM="C003"
```

## 原因

由於最後一個 Join 是 **一對多** 的關係，每個 `SoaexpandPartsResult` 會對應到多筆 `SoaBomSmallExplTempV` 資料，所以最終結果的筆數會等於所有符合條件的 `SoaBomSmallExplTempV` 資料筆數。

這是 LINQ Join 的正常行為，類似 SQL 的 INNER JOIN。


```sql
SELECT 
	h.WORK_NO, t.TX_ID, t.TOP_ITEM, 
	t.TOP_ITEM_DESC, t.PARENT_ITEM, 
	t.COMPONENT_ITEM, t.COMPONENT_ITEM_DESC, 
	t.COMPONENT_QUANTITY, t.IS_SUB 
FROM SoaCfgBomHeader h 
	INNER JOIN SoaexpandPartsResult p ON h.TX_ID = p.TX_ID 
	INNER JOIN SoaBomSmallExplTempV t ON p.TX_ID = t.TX_ID 
WHERE LTRIM(RTRIM(h.WORK_NO)) IN ('工單號1', '工單號2', '工單號3', ...) -- workNos 的內容 
	AND p.IsSuccess = 1 -- 或者 = 'true'，依資料型態而定
```
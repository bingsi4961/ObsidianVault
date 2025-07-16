---
date: 2025-07-16 10:50
aliases: 
tags: []
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
#### 📑 [[]]

---
## 1. 取得最大欄位編號

### 舊方法（手動迴圈）

```csharp
// 取得已使用的範圍
var usedRange = worksheet.RangeUsed();

// 計算最大的欄位編號
int maxColumn = 1;
for (int row = 1; row <= usedRange.RowCount(); row++)
{
    var lastCellUsed = worksheet.Row(row).LastCellUsed();
    if (lastCellUsed != null)
    {
        // lastCellUsed.Address.ColumnNumber 說明：
        // - lastCellUsed.Address 取得儲存格的位址資訊
        // - ColumnNumber 屬性回傳該儲存格的欄位編號（數字）
        // - 例如：A欄 = 1, B欄 = 2, C欄 = 3 ... Z欄 = 26, AA欄 = 27
        // - 這樣可以用數字來比較和計算欄位位置
        maxColumn = Math.Max(maxColumn, lastCellUsed.Address.ColumnNumber);
    }
}
```

實際應用情境，假設你有一個 Excel 檔案：

- 第1列：A1 到 E1 有資料
- 第2列：A2 到 C2 有資料
- 第3列：A3 到 G3 有資料

執行後 `maxColumn` 會是 **7**（對應 G 欄），因為 G 欄是所有列中最右邊有資料的欄位。

### 新方法（直接取得）

```csharp
// 方法1：直接從 RangeUsed() 取得
var usedRange = worksheet.RangeUsed();
int maxColumn = usedRange.LastColumn().ColumnNumber();

// 方法2：使用 RangeAddress
int maxColumn = worksheet.RangeUsed().RangeAddress.LastAddress.ColumnNumber;
```

## 2. RangeAddress 相關屬性

### 基本概念

- **RangeAddress**: 表示範圍位址資訊的物件，包含起始和結束位置
- **FirstAddress**: 範圍的起始位置（左上角儲存格）
- **LastAddress**: 範圍的結束位置（右下角儲存格）

### 實際範例

假設資料分佈在 A1 到 E10：

```csharp
var usedRange = worksheet.RangeUsed();
var rangeAddress = usedRange.RangeAddress;

// FirstAddress - 起始位置 (A1)
Console.WriteLine($"起始欄位: {rangeAddress.FirstAddress.ColumnNumber}"); // 1 (A欄)
Console.WriteLine($"起始列數: {rangeAddress.FirstAddress.RowNumber}");    // 1
Console.WriteLine($"起始位址: {rangeAddress.FirstAddress}");              // A1

// LastAddress - 結束位置 (E10)
Console.WriteLine($"結束欄位: {rangeAddress.LastAddress.ColumnNumber}");  // 5 (E欄)
Console.WriteLine($"結束列數: {rangeAddress.LastAddress.RowNumber}");     // 10
Console.WriteLine($"結束位址: {rangeAddress.LastAddress}");               // E10

// 完整範圍資訊
Console.WriteLine($"範圍: {rangeAddress}");                              // A1:E10
```

## 3. 常用屬性

```csharp
var address = rangeAddress.FirstAddress; // 或 LastAddress

// 取得位置資訊
int columnNumber = address.ColumnNumber;    // 欄位編號 (1, 2, 3...)
int rowNumber = address.RowNumber;          // 列編號 (1, 2, 3...)
string columnLetter = address.ColumnLetter; // 欄位字母 (A, B, C...)
```

## 4. 其他相關方法

```csharp
var usedRange = worksheet.RangeUsed();

// 取得最大列數
int maxRow = usedRange.LastRow().RowNumber();

// 取得範圍資訊
var rangeAddress = usedRange.RangeAddress;
int firstColumn = rangeAddress.FirstAddress.ColumnNumber;
int lastColumn = rangeAddress.LastAddress.ColumnNumber;
int firstRow = rangeAddress.FirstAddress.RowNumber;
int lastRow = rangeAddress.LastAddress.RowNumber;
```

## 5. 實用場景

### 計算範圍大小

```csharp
int totalColumns = rangeAddress.LastAddress.ColumnNumber - rangeAddress.FirstAddress.ColumnNumber + 1;
int totalRows = rangeAddress.LastAddress.RowNumber - rangeAddress.FirstAddress.RowNumber + 1;
```

## 6. 欄位編號對應表

- A欄 = 1
- B欄 = 2
- C欄 = 3
- ...
- Z欄 = 26
- AA欄 = 27
- AB欄 = 28

## 7. 注意事項

- 使用 `RangeUsed()` 只會取得有資料的範圍，空白區域不會被包含
- 可以精確掌握 Excel 資料的範圍，避免處理到空白區域
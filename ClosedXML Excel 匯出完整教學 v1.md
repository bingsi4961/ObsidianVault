---
date: 2025-07-15 23:47
aliases: 
tags:
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
#### 📑 [[]]

---
## 1. 環境準備

### 1.1 必要的 using 宣告

```csharp
using ClosedXML.Excel;
using System.IO;
using System.ComponentModel;
using System.Linq;
```

## 2. 基本 Excel 操作

### 2.1 建立 Workbook 和 Worksheet（含正確的資源管理）

```csharp
public IActionResult ExportToExcel(int id) 
{
	using (var workbook = new XLWorkbook())
	using (var ms = new MemoryStream())
	{
	    var worksheet = workbook.Worksheets.Add("工作表名稱");
	    
	    // 操作 worksheet
	    
	    // 儲存到 MemoryStream
	    workbook.SaveAs(ms);
	    ms.Flush();
	    ms.Position = 0;
	    
	    // 或儲存到檔案
	    // workbook.SaveAs("檔案路徑.xlsx");
	    
	    string fileName = DateTime.Now.ToString("yyyyMMddHHmmssfff") + "_Export.xlsx"; 
	    return File(ms.ToArray(), "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", fileName);
	}
}
```

### 2.2 基本儲存格操作

```csharp
// 兩個版本通用
var worksheet = workbook.Worksheets.Add("資料表");

// 設定單一儲存格值
worksheet.Cell(1, 1).Value = "標題";
worksheet.Cell(1, 2).Value = 123;
worksheet.Cell(1, 3).Value = DateTime.Now;

// 使用字母座標
worksheet.Cell("A1").Value = "A1儲存格";
worksheet.Cell("B1").Value = "B1儲存格";

// 取得儲存格值
var cellValue = worksheet.Cell(1, 1).Value;
```

## 3. 進階文字格式化

### 3.1 RichText 部分文字加粗

```csharp
// 兩個版本通用
var cell = worksheet.Cell(1, 1);

// 基本用法
cell.RichText.AddText("群組：").SetBold(true);
cell.RichText.AddText("實際群組名稱").SetBold(false);

// 進階樣式控制
cell.RichText.AddText("重要：")
    .SetBold(true)
    .SetFontColor(XLColor.Red)
    .SetFontSize(12);

cell.RichText.AddText("一般文字")
    .SetBold(false)
    .SetFontColor(XLColor.Black)
    .SetFontSize(10);
```

### 3.2 儲存格內換行

```csharp
// 兩個版本通用
var cell = worksheet.Cell(1, 1);

// 方法一：使用 Environment.NewLine
string cellValue = $"第一行{Environment.NewLine}第二行{Environment.NewLine}第三行";

// 方法二：使用 \n（較簡潔）
string cellValue2 = "第一行\n第二行\n第三行";

cell.Value = cellValue;

// 重要：必須設定 WrapText 為 true
cell.Style.Alignment.WrapText = true;
```

## 4. 樣式設定

### 4.1 字體樣式

```csharp
// 兩個版本通用
var cell = worksheet.Cell(1, 1);

// 字體設定
cell.Style.Font.Bold = true;
cell.Style.Font.FontSize = 14;
cell.Style.Font.FontColor = XLColor.Blue;
cell.Style.Font.FontName = "Arial";

// 背景色
cell.Style.Fill.BackgroundColor = XLColor.Yellow;
cell.Style.Fill.BackgroundColor = XLColor.FromArgb(255, 255, 0); // 自訂顏色
```

### 4.2 對齊方式

```csharp
// 兩個版本通用
var cell = worksheet.Cell(1, 1);

// 水平對齊
cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Left;
cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Right;

// 垂直對齊
cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Center;
cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Top;
cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Bottom;
```

### 4.3 邊框設定

```csharp
// 兩個版本通用
var cell = worksheet.Cell(1, 1);

// 單一儲存格邊框
cell.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
cell.Style.Border.InsideBorder = XLBorderStyleValues.Thin;

// 範圍邊框
var range = worksheet.Range("A1:C3");
range.Style.Border.OutsideBorder = XLBorderStyleValues.Thick;
range.Style.Border.InsideBorder = XLBorderStyleValues.Thin;
```

## 5. 範圍操作

### 5.1 建立和操作範圍

```csharp
// 兩個版本通用
// 建立範圍
var range1 = worksheet.Range("A1:C3");
var range2 = worksheet.Range(1, 1, 3, 3); // 列1行1 到 列3行3

// 設定範圍值
range1.Value = "統一值";

// 批次設定樣式
range1.Style.Font.Bold = true;
range1.Style.Fill.BackgroundColor = XLColor.LightGray;
```

### 5.2 合併儲存格

```csharp
// 兩個版本通用
// 合併指定範圍
worksheet.Range("A1:C1").Merge();
worksheet.Range(1, 1, 1, 3).Merge();

// 條件式合併
if (itemCount > 1)
{
    worksheet.Range(startRow, 1, startRow + itemCount - 1, 1).Merge();
}
```

### 5.3 動態範圍計算和 lastCellUsed.Address.ColumnNumber 說明

```csharp
// 兩個版本通用
// 取得已使用的範圍
var usedRange = worksheet.RangeUsed();

// 計算最大欄位
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

// 使用計算出的範圍設定樣式
var dataRange = worksheet.Range(1, 1, usedRange.RowCount(), maxColumn);
dataRange.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
```

## 6. 列和欄操作

### 6.1 設定列高和欄寬

```csharp
// 兩個版本通用
// 設定特定列高
worksheet.Row(1).Height = 25;
worksheet.Rows(1, 3).Height = 20;

// 設定預設列高
worksheet.RowHeight = 20;

// 自動調整欄寬
worksheet.Columns().AdjustToContents();

// 設定特定欄寬
worksheet.Column(1).Width = 15;
worksheet.Columns("A:C").Width = 20;
```

### 6.2 隱藏列或欄

```csharp
// 兩個版本通用
// 隱藏列
worksheet.Row(2).Hide();

// 隱藏欄
worksheet.Column(2).Hide();

// 顯示隱藏的列或欄
worksheet.Row(2).Unhide();
worksheet.Column(2).Unhide();
```

## 7. 資料填入技巧

### 7.1 使用反射自動填入資料

```csharp
// 兩個版本通用
using (var workbook = new XLWorkbook())
using (var ms = new MemoryStream())
{
    var worksheet = workbook.Worksheets.Add("資料表");
    
    if (data.Any())
    {
        // 取得屬性
        var properties = typeof(T).GetProperties();
        
        // 建立標題列
        for (int i = 0; i < properties.Length; i++)
        {
            var property = properties[i];
            
            // 取得 DisplayName 屬性
            string displayName = property.Name;
            var displayNameAttr = property.GetCustomAttribute(typeof(DisplayNameAttribute)) as DisplayNameAttribute;
            if (displayNameAttr != null)
            {
                displayName = displayNameAttr.DisplayName;
            }
            
            worksheet.Cell(1, i + 1).Value = displayName;
            worksheet.Cell(1, i + 1).Style.Font.Bold = true;
        }
        
        // 填入資料
        for (int rowIndex = 0; rowIndex < data.Count; rowIndex++)
        {
            var item = data[rowIndex];
            
            for (int colIndex = 0; colIndex < properties.Length; colIndex++)
            {
                var value = properties[colIndex].GetValue(item);
                worksheet.Cell(rowIndex + 2, colIndex + 1).Value = value?.ToString() ?? "";
            }
        }
    }
    
    workbook.SaveAs(ms);
    ms.Flush();
    ms.Position = 0;
}
```

## 8. 實際應用範例

### 8.1 建立標題區塊

```csharp
// 兩個版本通用
using (var workbook = new XLWorkbook())
using (var ms = new MemoryStream())
{
    var worksheet = workbook.Worksheets.Add("報表");
    
    // 使用 RichText 建立資訊區塊
    var cell1 = worksheet.Cell(1, 1);
    cell1.RichText.AddText("群組：").SetBold(true);
    cell1.RichText.AddText("實際群組名稱").SetBold(false);
    
    var cell2 = worksheet.Cell(2, 1);
    cell2.RichText.AddText("Stage：").SetBold(true);
    cell2.RichText.AddText("實際Stage名稱").SetBold(false);
    
    var cell3 = worksheet.Cell(3, 1);
    cell3.RichText.AddText("Device：").SetBold(true);
    cell3.RichText.AddText("實際Device名稱").SetBold(false);
    
    // 合併儲存格
    worksheet.Range(1, 1, 1, 3).Merge();
    worksheet.Range(2, 1, 2, 3).Merge();
    worksheet.Range(3, 1, 3, 3).Merge();
    
    workbook.SaveAs(ms);
    ms.Flush();
    ms.Position = 0;
}
```

### 8.2 建立表格邊框

```csharp
// 兩個版本通用
using (var workbook = new XLWorkbook())
using (var ms = new MemoryStream())
{
    var worksheet = workbook.Worksheets.Add("表格");
    
    int startRow = 1, endRow = 5, startCol = 1, endCol = 4;
    var range = worksheet.Range(startRow, startCol, endRow, endCol);
    
    // 外框粗線
    range.Style.Border.OutsideBorder = XLBorderStyleValues.Thick;
    
    // 內框細線
    range.Style.Border.InsideBorder = XLBorderStyleValues.Thin;
    
    // 標題列特殊樣式
    var headerRange = worksheet.Range(startRow, startCol, startRow, endCol);
    headerRange.Style.Font.Bold = true;
    headerRange.Style.Fill.BackgroundColor = XLColor.LightGray;
    headerRange.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
    
    workbook.SaveAs(ms);
    ms.Flush();
    ms.Position = 0;
}
```

### 8.3 處理多行文字

```csharp
// 兩個版本通用
using (var workbook = new XLWorkbook())
using (var ms = new MemoryStream())
{
    var worksheet = workbook.Worksheets.Add("多行文字");
    
    var lines = new List<string> { "第一行", "第二行", "第三行" };
    var cell = worksheet.Cell(1, 1);
    
    // 合併所有行
    string cellValue = string.Join("\n", lines);
    cell.Value = cellValue;
    
    // 啟用自動換行
    cell.Style.Alignment.WrapText = true;
    
    // 垂直置中
    cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Center;
    
    workbook.SaveAs(ms);
    ms.Flush();
    ms.Position = 0;
}
```

### 8.4 完整匯出範例

```csharp
// 兩個版本通用
using (var workbook = new XLWorkbook())
using (var ms = new MemoryStream())
{
    var worksheet = workbook.Worksheets.Add("會員清單");
    
    // 建立標題
    var headers = new[] { "編號", "姓名", "電子郵件", "建立日期" };
    for (int i = 0; i < headers.Length; i++)
    {
        var cell = worksheet.Cell(1, i + 1);
        cell.Value = headers[i];
        cell.Style.Font.Bold = true;
        cell.Style.Font.FontSize = 12;
        cell.Style.Fill.BackgroundColor = XLColor.FromArgb(220, 220, 220);
        cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
        cell.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
    }
    
    // 填入資料
    for (int i = 0; i < members.Count; i++)
    {
        var member = members[i];
        var row = i + 2;
        
        worksheet.Cell(row, 1).Value = member.Id;
        worksheet.Cell(row, 2).Value = member.Name;
        worksheet.Cell(row, 3).Value = member.Email;
        worksheet.Cell(row, 4).Value = member.CreateDate.ToString("yyyy/MM/dd");
        
        // 套用資料樣式
        for (int col = 1; col <= 4; col++)
        {
            var cell = worksheet.Cell(row, col);
            cell.Style.Font.FontSize = 10;
            cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Left;
            cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Center;
            cell.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
        }
    }
    
    // 調整欄寬
    worksheet.Columns().AdjustToContents();
    
    workbook.SaveAs(ms);
    ms.Flush();
    ms.Position = 0;
}
```
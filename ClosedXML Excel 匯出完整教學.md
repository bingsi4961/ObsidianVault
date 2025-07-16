---
date: 2025-07-15 23:47
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

		// 方式一：直接下載（推薦）
		string fileName = DateTime.Now.ToString("yyyyMMddHHmmssfff") + "_Export.xlsx"; 
	    return File(ms.ToArray(), "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", fileName);
	    
	    // 方式二：儲存到伺服器 
		string folderPath = @"C:\Exports\"; 
		string fileName = $"Export_{DateTime.Now:yyyyMMddHHmmss}.xlsx"; 
		string fullPath = Path.Combine(folderPath, fileName); 
		Directory.CreateDirectory(folderPath); 
		workbook.SaveAs(fullPath);

		// 方式三：Base64 編碼
		string base64Result = Convert.ToBase64String(ms.ToArray()); 
		var xls = new MemoryStream(Convert.FromBase64String(base64Result)); 
		return File(xls.ToArray(), "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet", fileName);
	}
}
```

### 2.2 基本儲存格操作

```csharp
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
// 合併指定範圍
worksheet.Range("A1:C1").Merge();
worksheet.Range(1, 1, 1, 3).Merge();
```

### 5.3 動態範圍計算和 lastCellUsed.Address.ColumnNumber 說明

##### 📑 [[ClosedXML 範圍處理筆記]]

![[ClosedXML 範圍處理筆記]]

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

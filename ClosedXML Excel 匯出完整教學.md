---
date : 2025-07-15 18:44
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
#### 📑 [[]]

---
### 1.3 必要的 using 宣告

```csharp
using ClosedXML.Excel;
using System.IO;
using System.ComponentModel;
using System.Linq;
```

## 2. 基本 Excel 操作

### 2.1 建立 Workbook 和 Worksheet

```csharp
// 兩個版本通用
using (var workbook = new XLWorkbook())
{
    var worksheet = workbook.Worksheets.Add("工作表名稱");
    
    // 操作 worksheet
    
    // 儲存到檔案
    workbook.SaveAs("檔案路徑.xlsx");
    
    // 或儲存到 MemoryStream
    var ms = new MemoryStream();
    workbook.SaveAs(ms);
    ms.Flush();
    ms.Position = 0;
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

### 5.3 動態範圍計算

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
        maxColumn = Math.Max(maxColumn, lastCellUsed.Address.ColumnNumber);
    }
}

// 對整個使用範圍設定樣式
usedRange.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
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
public void FillDataWithReflection<T>(IXLWorksheet worksheet, List<T> data)
{
    if (!data.Any()) return;
    
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
```

### 7.2 複雜資料結構填入

```csharp
// 兩個版本通用
public void FillComplexData(IXLWorksheet worksheet, List<CategoryData> categories)
{
    int currentRow = 1;
    
    foreach (var category in categories)
    {
        // 類別標題
        var categoryCell = worksheet.Cell(currentRow, 1);
        categoryCell.Value = category.Name;
        categoryCell.Style.Font.Bold = true;
        categoryCell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
        
        var items = category.Items ?? new List<ItemData>();
        
        if (items.Any())
        {
            // 如果有多個項目，合併類別儲存格
            if (items.Count > 1)
            {
                worksheet.Range(currentRow, 1, currentRow + items.Count - 1, 1).Merge();
            }
            
            // 填入項目資料
            foreach (var item in items)
            {
                worksheet.Cell(currentRow, 2).Value = item.Name;
                worksheet.Cell(currentRow, 3).Value = item.Value;
                currentRow++;
            }
        }
        else
        {
            currentRow++;
        }
    }
}
```

## 8. 實用方法集合

### 8.1 建立標題區塊

```csharp
// 兩個版本通用
public void CreateInfoBlock(IXLWorksheet worksheet, string groupName, string stageName, string deviceName)
{
    // 使用 RichText 建立資訊區塊
    var cell1 = worksheet.Cell(1, 1);
    cell1.RichText.AddText("群組：").SetBold(true);
    cell1.RichText.AddText(groupName).SetBold(false);
    
    var cell2 = worksheet.Cell(2, 1);
    cell2.RichText.AddText("Stage：").SetBold(true);
    cell2.RichText.AddText(stageName).SetBold(false);
    
    var cell3 = worksheet.Cell(3, 1);
    cell3.RichText.AddText("Device：").SetBold(true);
    cell3.RichText.AddText(deviceName).SetBold(false);
    
    // 合併儲存格
    worksheet.Range(1, 1, 1, 3).Merge();
    worksheet.Range(2, 1, 2, 3).Merge();
    worksheet.Range(3, 1, 3, 3).Merge();
}
```

### 8.2 建立表格邊框

```csharp
// 兩個版本通用
public void CreateTableBorder(IXLWorksheet worksheet, int startRow, int endRow, int startCol, int endCol)
{
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
}
```

### 8.3 處理多行文字

```csharp
// 兩個版本通用
public void CreateMultiLineCell(IXLWorksheet worksheet, int row, int col, List<string> lines)
{
    var cell = worksheet.Cell(row, col);
    
    // 合併所有行
    string cellValue = string.Join("\n", lines);
    cell.Value = cellValue;
    
    // 啟用自動換行
    cell.Style.Alignment.WrapText = true;
    
    // 垂直置中
    cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Center;
}
```

## 9. 顏色和樣式常數

### 9.1 常用顏色定義

```csharp
// 兩個版本通用
public static class ExcelColors
{
    public static XLColor HeaderBackground = XLColor.FromArgb(220, 220, 220);
    public static XLColor ImportantText = XLColor.Red;
    public static XLColor NormalText = XLColor.Black;
    public static XLColor WarningBackground = XLColor.Yellow;
    public static XLColor SuccessBackground = XLColor.FromArgb(200, 255, 200);
}
```

### 9.2 標準樣式模板

```csharp
// 兩個版本通用
public static class ExcelStyles
{
    public static void ApplyHeaderStyle(IXLCell cell)
    {
        cell.Style.Font.Bold = true;
        cell.Style.Font.FontSize = 12;
        cell.Style.Fill.BackgroundColor = ExcelColors.HeaderBackground;
        cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
        cell.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
    }
    
    public static void ApplyDataStyle(IXLCell cell)
    {
        cell.Style.Font.FontSize = 10;
        cell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Left;
        cell.Style.Alignment.Vertical = XLAlignmentVerticalValues.Center;
        cell.Style.Border.OutsideBorder = XLBorderStyleValues.Thin;
    }
}
```

## 10. 記憶體管理和效能

### 10.1 正確的資源管理

```csharp
// 兩個版本通用
public MemoryStream CreateExcelFile(List<DataModel> data)
{
    using (var workbook = new XLWorkbook())
    {
        var worksheet = workbook.Worksheets.Add("資料表");
        
        // 處理資料
        ProcessData(worksheet, data);
        
        // 建立 MemoryStream
        var ms = new MemoryStream();
        workbook.SaveAs(ms);
        ms.Flush();
        ms.Position = 0;
        
        // workbook 會自動 dispose
        return ms;
    }
}
```

### 10.2 大量資料處理建議

```csharp
// 兩個版本通用，但 0.95.4 版本效能更好
public void ProcessLargeData(IXLWorksheet worksheet, List<DataModel> data)
{
    // 0.95.2 版本：建議超過 5000 筆分批處理
    // 0.95.4 版本：可以處理更大量資料
    
    const int batchSize = 1000;
    
    for (int i = 0; i < data.Count; i += batchSize)
    {
        var batch = data.Skip(i).Take(batchSize).ToList();
        ProcessBatch(worksheet, batch, i);
    }
}
```

## 11. 完整範例

### 11.1 標準匯出範例

```csharp
// 兩個版本通用
public MemoryStream ExportToExcel(List<MemberModel> members)
{
    using (var workbook = new XLWorkbook())
    {
        var worksheet = workbook.Worksheets.Add("會員清單");
        
        // 建立標題
        var headers = new[] { "編號", "姓名", "電子郵件", "建立日期" };
        for (int i = 0; i < headers.Length; i++)
        {
            var cell = worksheet.Cell(1, i + 1);
            cell.Value = headers[i];
            ExcelStyles.ApplyHeaderStyle(cell);
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
                ExcelStyles.ApplyDataStyle(worksheet.Cell(row, col));
            }
        }
        
        // 調整欄寬
        worksheet.Columns().AdjustToContents();
        
        // 建立並返回 MemoryStream
        var ms = new MemoryStream();
        workbook.SaveAs(ms);
        ms.Flush();
        ms.Position = 0;
        
        return ms;
    }
}
```

### 11.2 複雜格式範例

```csharp
// 兩個版本通用
public MemoryStream ExportComplexReport(ReportData reportData)
{
    using (var workbook = new XLWorkbook())
    {
        var worksheet = workbook.Worksheets.Add("報表");
        
        // 報表標題
        var titleCell = worksheet.Cell(1, 1);
        titleCell.Value = "月度報表";
        titleCell.Style.Font.Bold = true;
        titleCell.Style.Font.FontSize = 16;
        worksheet.Range(1, 1, 1, 5).Merge();
        titleCell.Style.Alignment.Horizontal = XLAlignmentHorizontalValues.Center;
        
        // 資訊區塊
        CreateInfoBlock(worksheet, reportData.Department, reportData.Month, reportData.Year);
        
        // 資料表格
        int startRow = 5;
        var headers = new[] { "項目", "數量", "金額", "備註" };
        
        for (int i = 0; i < headers.Length; i++)
        {
            var cell = worksheet.Cell(startRow, i + 1);
            cell.Value = headers[i];
            ExcelStyles.ApplyHeaderStyle(cell);
        }
        
        // 填入明細資料
        for (int i = 0; i < reportData.Items.Count; i++)
        {
            var item = reportData.Items[i];
            var row = startRow + i + 1;
            
            worksheet.Cell(row, 1).Value = item.Name;
            worksheet.Cell(row, 2).Value = item.Quantity;
            worksheet.Cell(row, 3).Value = item.Amount;
            worksheet.Cell(row, 4).Value = item.Remark;
            
            // 多行備註處理
            if (!string.IsNullOrEmpty(item.Remark) && item.Remark.Contains("\n"))
            {
                worksheet.Cell(row, 4).Style.Alignment.WrapText = true;
            }
        }
        
        // 建立表格邊框
        CreateTableBorder(worksheet, startRow, startRow + reportData.Items.Count, 1, 4);
        
        // 調整欄寬
        worksheet.Columns().AdjustToContents();
        
        var ms = new MemoryStream();
        workbook.SaveAs(ms);
        ms.Flush();
        ms.Position = 0;
        
        return ms;
    }
}
```

這個重新整理的教學專注於 ClosedXML 的核心功能，移除了不相關的程式碼，並明確標示所有範例都是兩個版本通用的。您可以直接使用這些範例來學習 ClosedXML 的各種功能。
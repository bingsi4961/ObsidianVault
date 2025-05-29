---
title: 從 ViewModel 產生 Excel 檔
tags: ['C# 程式']

---

```csharp=
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Configuration;
using System.Reflection;
using ClosedXML.Excel;

/// <summary>
/// 歷史記錄模型類別
/// </summary>
public class History
{
    [DisplayName("ActionTable")]
    public string actionTable { get; set; }
    
    [DisplayName("Action")]
    public string Action { get; set; }
    
    [DisplayName("PLM")]
    public string PLM { get; set; }
    
    [DisplayName("ColumnName")]
    public string ColumnName { get; set; }
    
    [DisplayName("Before")]
    public string Before { get; set; }
    
    [DisplayName("After")]
    public string After { get; set; }
    
    [DisplayName("ModifyUser")]
    public string ModifyUser { get; set; }
    
    [DisplayName("ModifyDate")]
    public string ModifyDate { get; set; }
   
    // 查詢參數
    public string sDate { get; set; }
    public string eDate { get; set; }
    public int year { get; set; }
    public string category { get; set; }
    public string software { get; set; }
    public string productLine { get; set; }
    public int TableCodeId { get; set; }
}

/// <summary>
/// 資料存取類別
/// </summary>
public class SWDao
{
    private readonly MSSQLAccess _db = new MSSQLAccess();
    
    /// <summary>
    /// 連線字串
    /// </summary>
    public string ConnString 
    { 
        get => _db.ConnString;
        set => _db.ConnString = value;
    }
    
    /// <summary>
    /// 取得一週內MKT規格變更紀錄
    /// </summary>
    /// <param name="sDate">開始日期</param>
    /// <param name="eDate">結束日期</param>
    /// <returns>變更歷史記錄列表</returns>
    public List<History> GetMKTSpecChangesWeekly(string sDate, string eDate)
    {
        var param = new History
        {
            sDate = sDate,
            eDate = eDate
        };
        
        string sql = @"SELECT 'MKT Name' AS actionTable
                        ,actionCode.CodeValue AS [Action]
                        ,'' AS PLM
                        ,mlog.ColumnName
                        ,mlog.ColumnValue AS [Before]
                        ,mlog.AfterValue AS [After]
                        ,mlog.ModifyUser
                        ,mlog.ModifyDate
                    FROM ModifyLog mlog						
                    ORDER BY mlog.ModifyDate DESC";
                    
        _db.Cmd = sql;
        return _db.ExcuteQuery<History, History>(param);
    }
    
    /// <summary>
    /// 取得MKT規格歷史記錄
    /// </summary>
    /// <param name="sDate">開始日期</param>
    /// <param name="eDate">結束日期</param>
    /// <param name="year">年份</param>
    /// <returns>MKT規格歷史記錄列表</returns>
    public List<History> GetMKTSpecHistory(string sDate, string eDate, int year)
    {
        // 實作內容省略
        return new List<History>();
    }
}

/// <summary>
/// 通知處理類別
/// </summary>
public class NotificationHandler
{
    /// <summary>
	/// 發送MKT規格變更週報通知並產生Excel報表
	/// </summary>
	public void Notification()
	{
		// 1. 設定時間參數 - 報表期間為過去一週
		string sDate = DateTime.Now.AddDays(-7).ToString("yyyy-MM-dd 12:00:00");
		string eDate = DateTime.Now.ToString("yyyy-MM-dd 12:00:00");
		int year = DateTime.Now.AddDays(-7).Year - 1;
		
		// 2. 初始化資料存取物件
		var dao = new SWDao() { 
			ConnString = ConfigurationManager.ConnectionStrings["MSSQLConn"].ConnectionString 
		};
		
		// 3. 收集異動資料
		// 與SW Mapping相關的MKT規格異動
		List<History> swMappingChanges = dao.GetMKTSpecHistory(sDate, eDate, year);
		
		// 所有MKT規格異動項目
		List<History> allSpecChanges = dao.GetMKTSpecChangesWeekly(sDate, eDate);
		
		// 4. 準備Excel工作表資料
		var sheetsData = new Dictionary<string, List<History>>();
		
		// 只加入有資料的工作表
		if (swMappingChanges.Count > 0) { 
			sheetsData.Add("SW Mapping Items Change", swMappingChanges); 
		}
		
		if (allSpecChanges.Count > 0) { 
			sheetsData.Add("MKT Spec All Items Change", allSpecChanges); 
		}
		
		// 5. 如果有資料，產生Excel報表
		if (sheetsData.Count > 0)
		{
			try
			{
				// 建立報表文件名稱
				string fileName = $"MKT Spec異動_{DateTime.Now.ToString("yyyyMMdd")}.xlsx";
				string fullPath = ConfigurationManager.AppSettings["ExcelPath"] + fileName;
				
				// 產生Excel報表
				using (XLWorkbook workbook = new XLWorkbook())
				{
					// 填入資料到Excel
					this.SetData(sheetsData, workbook);
					
					// 儲存Excel檔案
					workbook.SaveAs(fullPath);
				}
				
				// 此處可以加入發送通知邏輯
				// SendNotification(fullPath, sheetsData);
			}
			catch (Exception ex)
			{
				// 記錄例外狀況
				LogError("產生MKT Spec異動報表失敗", ex);
				throw;
			}
		}
	}
    
    /// <summary>
    /// 將資料填入Excel工作表
    /// </summary>
    /// <param name="sheetsData">工作表資料字典</param>
    /// <param name="workbook">Excel工作簿</param>
    private void SetData(Dictionary<string, List<History>> sheetsData, XLWorkbook workbook)
    {
        foreach (var sheetData in sheetsData)
        {
            string sheetName = sheetData.Key;
            List<History> data = sheetData.Value;
            
            // 建立新工作表
            var sheet = workbook.Worksheets.Add(sheetName);
            
            // 欄位起始位置
            int rowIdx = 1;
            int colIdx = 1;
            
            // 使用 reflection 設定工作表標題
            foreach (var property in typeof(History).GetProperties())
            {
                // 取得顯示名稱屬性
                DisplayNameAttribute display = property.GetCustomAttribute(typeof(DisplayNameAttribute)) as DisplayNameAttribute;
                
                if (display != null)
                {
                    // 設定標題格式
                    var cell = sheet.Cell(rowIdx, colIdx);
                    cell.Style.Fill.BackgroundColor = XLColor.FromArgb(163, 208, 255);
                    cell.Style.Border.TopBorder = XLBorderStyleValues.Thin;
                    cell.Style.Border.BottomBorder = XLBorderStyleValues.Thin;
                    cell.Style.Border.RightBorder = XLBorderStyleValues.Thin;
                    cell.Style.Border.LeftBorder = XLBorderStyleValues.Thin;
                    cell.Value = display.DisplayName;
                    
                    colIdx++;
                }
            }
            
            // 填充資料 (從第二列開始)
            rowIdx = 2;
            foreach (var item in data)
            {
                int columnIndex = 1;
                
                foreach (var property in item.GetType().GetProperties())
                {
                    DisplayNameAttribute display = property.GetCustomAttribute(typeof(DisplayNameAttribute)) as DisplayNameAttribute;
                    
                    if (display != null)
                    {
                        // 取得屬性值並填入儲存格
                        object value = property.GetValue(item, null);
                        sheet.Cell(rowIdx, columnIndex).Value = Convert.ToString(value ?? "");
                        columnIndex++;
                    }
                }
                
                rowIdx++;
            }
            
            // 調整欄寬
            sheet.Columns(1, colIdx).AdjustToContents();
        }
    }
}
```

# Notification() 方法解說

## 設定時間參數
```csharp
int year = DateTime.Now.AddDays(-7).Year - 1;
```
這行計算一個特殊的年份參數：
1. 先取得7天前的日期（`DateTime.Now.AddDays(-7)`）
2. 從該日期取出年份（`.Year`）
3. 再減去1年
- 例如：如果今天是2025-02-20，則7天前是2025-02-13，其年份是2025，減1後得到2024
- 這可能用於取得去年同期的歷史資料進行比較

## 產生 Excel 報表


```csharp
using (XLWorkbook workbook = new XLWorkbook())
{
```
建立新的 Excel 工作簿物件，使用 `using` 陳述式確保資源會被正確釋放：
- `ClosedXML` 是處理 Excel 檔案的第三方函式庫
- `XLWorkbook` 是該庫中表示 Excel 檔案的主要類別

```csharp
this.SetData(sheetsData, workbook);
```
呼叫 `SetData` 方法（稍後會詳細解析）：
- 傳入準備好的工作表資料（`sheetsData`）
- 以及新建立的工作簿物件（`workbook`）
- 這個方法負責將資料填入工作表中

# SetData() 方法解說

## 方法開始
```csharp
private void SetData(Dictionary<string, List<History>> sheetsData, XLWorkbook workbook)
{
```
接受兩個參數：
  1. `sheetsData`：包含各工作表名稱和對應資料的字典
  2. `workbook`：要填入資料的 Excel 工作簿物件

## 迭代每個工作表
```csharp
foreach (var sheetData in sheetsData)
{
```
使用 foreach 迴圈遍歷 `sheetsData` 字典中的每一項：
- 每次迭代，`sheetData` 代表一個鍵值對（工作表名稱和資料）

```csharp
string sheetName = sheetData.Key;
List<History> data = sheetData.Value;
```
從當前迭代的鍵值對中取出：
- 工作表名稱（`sheetName`）
- 對應的資料清單（`data`）

## 建立工作表
```csharp
var sheet = workbook.Worksheets.Add(sheetName);
```
在工作簿中新增一個工作表：
- 使用前面取得的 `sheetName` 作為工作表名稱
- `workbook.Worksheets.Add()` 方法新增工作表並返回該工作表物件

## 初始化位置變數
```csharp
int rowIdx = 1;
int colIdx = 1;
```
設定起始位置變數：
- `rowIdx`：行索引，從1開始（Excel 使用1為第一行）
- `colIdx`：列索引，從1開始（Excel 使用1為第一列）

## 設定標題列
```csharp
foreach (var property in typeof(History).GetProperties())
{
```
使用反射（Reflection）機制：
1. `typeof(History)` 獲取 History 類別的型別資訊
2. `.GetProperties()` 取得該類別的所有公開屬性
3. 迭代每個屬性

```csharp
DisplayNameAttribute display = property.GetCustomAttribute(typeof(DisplayNameAttribute)) as DisplayNameAttribute;
```
檢查屬性是否有 `DisplayNameAttribute`：
1. `GetCustomAttribute` 方法獲取指定型別的自訂屬性
2. 嘗試將結果轉型為 `DisplayNameAttribute`
3. 若屬性沒有這個特性，`display` 會是 null

```csharp
if (display != null)
{
```
只處理有 `DisplayNameAttribute` 的屬性：
- 這確保只有標記為顯示名稱的屬性會被加入 Excel 表頭

```csharp
var cell = sheet.Cell(rowIdx, colIdx);
```
取得目前位置的儲存格：
- 使用當前的行索引 `rowIdx` 和列索引 `colIdx`
- 返回一個代表該儲存格的物件，可進行樣式和值的設定

```csharp
cell.Style.Fill.BackgroundColor = XLColor.FromArgb(163, 208, 255);
```
設定儲存格背景色：
- 使用 RGB 色彩模型 (163, 208, 255)，這是一種淡藍色
- 使標題列視覺上更容易識別

```csharp
cell.Style.Border.TopBorder = XLBorderStyleValues.Thin;
cell.Style.Border.BottomBorder = XLBorderStyleValues.Thin;
cell.Style.Border.RightBorder = XLBorderStyleValues.Thin;
cell.Style.Border.LeftBorder = XLBorderStyleValues.Thin;
```
設定儲存格四邊的邊框樣式：
- 所有四邊都設定為細線（`Thin`）
- 這使標題列看起來更加清晰，與資料區分開來

```csharp
cell.Value = display.DisplayName;
```
設定儲存格的值：
- 使用從 `DisplayNameAttribute` 取得的顯示名稱
- 這樣 Excel 表頭顯示的是更友善的名稱，而非原始屬性名稱

## 填充資料列
```csharp
rowIdx = 2;
```
重設行索引為2：
- 第1行已用於標題
- 從第2行開始填入實際資料

```csharp
foreach (var property in item.GetType().GetProperties())
{
```
同樣使用反射，遍歷當前物件的所有屬性：
- `item.GetType()` 獲取實際執行時期型別
- `.GetProperties()` 取得所有公開屬性

```csharp
DisplayNameAttribute display = property.GetCustomAttribute(typeof(DisplayNameAttribute)) as DisplayNameAttribute;
```
同樣檢查屬性是否有 `DisplayNameAttribute`：
- 確保只處理有顯示名稱的屬性，與標題列一致

```csharp
if (display != null)
{
```
只處理有 `DisplayNameAttribute` 的屬性：
- 保持資料與標題的一致性

```csharp
object value = property.GetValue(item, null);
```
使用反射獲取屬性值：
1. `property.GetValue(item, null)` 從當前物件取得屬性值
2. 第一個參數 `item` 是要取值的物件實例
3. 第二個參數 `null` 表示沒有索引參數（對於非索引屬性）

## 調整列寬
```csharp
sheet.Columns(1, colIdx).AdjustToContents();
```
自動調整列寬：
1. `sheet.Columns(1, colIdx)` 選取從第1列到最後使用的列
2. `.AdjustToContents()` 根據內容自動調整每列的寬度
3. 使Excel表格更易於閱讀，避免內容被截斷
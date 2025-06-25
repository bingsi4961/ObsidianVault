---
date: 2025-06-10 23:30
aliases: 
tags:
  - CSharp_語法
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## 取得年度 `SelectList`

```csharp
public SelectList GetYearSelectList(string selectedValue = "")
{
    var yearList = new List<SelectListItem>();
    int currentYear = DateTime.Now.Year;
    
    // 從 2023 年到明年 (今年+1)
    for (int year = 2023; year <= currentYear + 1; year++)
    {
        yearList.Add(new SelectListItem
        {
            Value = year.ToString(),
            Text = year.ToString()
        });
    }
    
    return new SelectList(yearList, "Value", "Text", selectedValue);
}
```

```csharp
public ActionResult Index()
{
    ViewBag.YearList = GetYearSelectList();
    return View();
}
```

```html
@Html.DropDownList("Year", ViewBag.YearList as SelectList, "請選擇年份", new { @class = "form-control" })
```

## `List<VendorModel>` 轉換成 `SelectList`

- Model.VendorList 的型態是 `List<VendorModel>`
- VendorModel 有屬性 VendorId、VendorName

```csharp
new SelectList(Model.VendorList, "VendorId", "VendorName", selectedValue);
```

```html
@Html.DropDownList("Vendor", new SelectList(Model.VendorList, "VendorId", "VendorName"), "請選擇廠商", new { @class = "form-control" })
```

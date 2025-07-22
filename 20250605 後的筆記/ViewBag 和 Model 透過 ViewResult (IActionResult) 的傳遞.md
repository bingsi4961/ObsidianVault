---
date: 2025-07-15 15:48
aliases: 
tags:
  - NetCore_MVC
  - MVC5
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[🛠️除了 ViewResult 之外，其他類似的 Result 類型]]
#### 📑 [[]]

---
## 核心觀念

### ViewBag 和 Model 的傳遞規則
- **直接回傳 `return FunctionB()`** → ViewBag 會直接傳遞 ✅
- **重新建立 View `return View(model)`** → ViewBag 不會傳遞，需要手動複製 ❌
- **Model 的部份** → 要看 MainFunction 的 return 傳什麼

### ViewBag 的兩種寫法（等價）
```csharp
// 方法 1：使用索引器
// var viewBag = viewResult.ViewData;
viewBag["AdditionalInfo"] = "這是從 MainFunction 新增的資訊";

// 方法 2：使用動態屬性
ViewBag.AdditionalInfo = "這是從 MainFunction 新增的資訊";
```

**重要差異：**
- `viewBag["AdditionalInfo"]` → 操作從 FunctionB 取得的 ViewData
- `ViewBag.AdditionalInfo` → 操作當前 Controller 的 ViewBag

### Pattern Matching 型別檢查和轉型
```csharp
// 同時做兩件事：檢查型別 + 轉型
if (result is ViewResult viewResult)
{
    // viewResult 已經是 ViewResult 型別
}

// 等同於傳統寫法：
if (result is ViewResult)
{
    var viewResult = (ViewResult)result;
}
```

---

## 完整程式碼範例

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ViewFeatures;

public class HomeController : Controller
{
    // 假設這是同事寫好的 functionB
    public IActionResult FunctionB()
    {
        // 建立 Model
        var model = new PersonModel
        {
            Name = "張三",
            Age = 30,
            Email = "zhang@example.com"
        };

        // 設定 ViewBag
        ViewBag.Title = "個人資料";
        ViewBag.Message = "歡迎使用系統";
        ViewBag.CurrentTime = DateTime.Now;

        return View(model);
    }

    // 方法 1：直接回傳 - ViewBag 會傳遞
    public IActionResult MainFunction1()
    {
        return FunctionB(); // ✅ ViewBag 直接傳遞
    }

    // 方法 2：取得後重新建立 View - ViewBag 不會傳遞
    public IActionResult MainFunction2()
    {
        var viewResult = FunctionB() as ViewResult;
        var model = viewResult.Model;
        
        return View(model); // ❌ ViewBag 不會傳遞，除非手動複製
    }

    // 方法 3：取得後重新建立 View，並手動複製 ViewBag
    public IActionResult MainFunction3()
    {
        var viewResult = FunctionB() as ViewResult;
        var model = viewResult.Model;
        
        // 手動複製 ViewBag
        foreach (var item in viewResult.ViewData)
        {
            ViewData[item.Key] = item.Value;
        }
        
        return View(model); // ✅ 現在 ViewBag 會傳遞
    }

    // 方法 4：使用 Pattern Matching 安全取得
    public IActionResult MainFunctionSafe()
    {
        var result = FunctionB();
        
        // 檢查是否為 ViewResult 並同時轉型
        if (result is ViewResult viewResult)
        {
            // 安全地取得 Model
            var model = viewResult.Model as PersonModel;
            
            // 取得 ViewBag (ViewData)
            var viewData = viewResult.ViewData;
            
            // 建立新的 ViewBag 給目前的 Action
            ViewBag.OriginalTitle = viewData["Title"];
            ViewBag.OriginalMessage = viewData["Message"];
            ViewBag.ProcessedBy = "MainFunction";
            
            // 可以將原始 Model 傳遞給新的 View
            return View("ProcessedView", model);
        }
        
        return View("Error");
    }

    // 方法 5：取得特定 ViewBag 項目的泛型方法
    public IActionResult MainFunctionGeneric()
    {
        var viewResult = FunctionB() as ViewResult;
        
        if (viewResult != null)
        {
            var model = viewResult.Model as PersonModel;
            var viewData = viewResult.ViewData;
            
            // 使用泛型方法安全地取得 ViewBag 值
            var title = GetViewBagValue<string>(viewData, "Title");
            var currentTime = GetViewBagValue<DateTime>(viewData, "CurrentTime");
            
            // 設定新的 ViewBag
            ViewBag.ProcessedTitle = $"處理後的標題: {title}";
            ViewBag.ProcessedTime = currentTime.ToString("yyyy-MM-dd HH:mm:ss");
            
            return View("FinalView", model);
        }
        
        return View();
    }

    // 輔助方法：安全地取得 ViewBag 值
    private T GetViewBagValue<T>(ViewDataDictionary viewData, string key)
    {
        if (viewData.ContainsKey(key) && viewData[key] is T value)
        {
            return value;
        }
        return default(T);
    }

    // 方法 6：如果您需要取得 ViewResult 的其他屬性
    public IActionResult MainFunctionAdvanced()
    {
        var viewResult = FunctionB() as ViewResult;
        
        if (viewResult != null)
        {
            // 取得 Model
            var model = viewResult.Model;
            
            // 取得 ViewData (包含 ViewBag)
            var viewData = viewResult.ViewData;
            
            // 取得 View 名稱
            var viewName = viewResult.ViewName;
            
            // 取得 TempData
            var tempData = viewResult.TempData;
            
            // 取得 ContentType
            var contentType = viewResult.ContentType;
            
            // 取得 StatusCode
            var statusCode = viewResult.StatusCode;
            
            Console.WriteLine($"View Name: {viewName}");
            Console.WriteLine($"Status Code: {statusCode}");
            Console.WriteLine($"Content Type: {contentType}");
            
            // 複製所有 ViewData 到當前的 ViewData
            foreach (var item in viewData)
            {
                ViewData[item.Key] = item.Value;
            }
            
            // 新增額外的 ViewBag 資訊
            ViewBag.ProcessedByAdvanced = true;
            ViewBag.ProcessingTime = DateTime.Now;
            
            return View("AdvancedView", model);
        }
        
        return View();
    }

    // 方法 7：修改從 FunctionB 取得的 ViewResult 後回傳
    public IActionResult MainFunctionModify()
    {
        var result = FunctionB() as ViewResult;
        
        if (result != null)
        {
            // 修改部分 ViewBag
            result.ViewData["Title"] = "修改後的標題";
            result.ViewData["AdditionalInfo"] = "MainFunction 新增的資訊";
            
            return result; // ✅ 修改後的 ViewBag 會傳遞
        }
        
        return View();
    }
}

// 範例 Model 類別
public class PersonModel
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Email { get; set; }
}
```

---

## 實際應用場景

### 共用資料準備邏輯
```csharp
public class HomeController : Controller
{
    public IActionResult GetUserData()
    {
        // 共用的資料準備邏輯
        var model = GetUserModel();
        ViewBag.Title = "使用者資料";
        ViewBag.LastLogin = DateTime.Now.AddDays(-1);
        
        return View(model);
    }

    // 直接使用 GetUserData 的結果
    public IActionResult UserProfile()
    {
        return GetUserData(); // ✅ 所有 ViewBag 都會傳遞
    }

    // 可以根據不同需求呼叫相同的資料準備邏輯
    public IActionResult UserSettings()
    {
        var result = GetUserData() as ViewResult;
        
        // 修改部分 ViewBag
        result.ViewData["Title"] = "使用者設定";
        result.ViewData["AdditionalInfo"] = "設定頁面";
        
        return result; // ✅ 修改後的 ViewBag 會傳遞
    }
}
```

### Pattern Matching 進階用法
```csharp
public IActionResult ProcessResult(IActionResult result)
{
    switch (result)
    {
        case ViewResult viewResult:
            // 直接使用 viewResult，已經是 ViewResult 型別
            var model = viewResult.Model;
            break;
            
        case JsonResult jsonResult:
            // 直接使用 jsonResult，已經是 JsonResult 型別
            var data = jsonResult.Value;
            break;
            
        case RedirectResult redirectResult:
            // 直接使用 redirectResult
            var url = redirectResult.Url;
            break;
            
        default:
            // 其他情況
            break;
    }
    
    return result;
}
```

---

## View 中的使用範例

```html
<!-- 如果使用直接回傳的方式，這些都可以正常顯示 -->
<h1>@ViewBag.Title</h1>
<p>@ViewBag.Message</p>
<p>時間：@ViewBag.CurrentTime</p>

<!-- Model 也可以正常使用 -->
<p>姓名：@Model.Name</p>
<p>年齡：@Model.Age</p>
<p>Email：@Model.Email</p>
```

---

## 重點整理

### ViewBag 傳遞機制
1. **直接回傳 `return FunctionB()`** → ViewBag 會直接傳遞 ✅
2. **重新建立 View `return View(model)`** → ViewBag 不會傳遞 ❌
3. **手動複製 ViewData** → ViewBag 會傳遞 ✅

### 最佳實務
- 如果不需要修改 ViewBag，直接回傳 FunctionB() 結果
- 如果需要修改或新增 ViewBag，手動複製 ViewData
- 使用 Pattern Matching 進行安全的型別檢查和轉型
- 使用泛型方法安全地取得 ViewBag 值

### 常見錯誤
- 以為 ViewBag 會自動傳遞到新建立的 View
- 混淆操作原始 ViewData 和當前 Controller 的 ViewBag
- 沒有進行型別檢查就直接轉型
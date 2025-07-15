---
date: 2025-07-15 17:16
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

```CSharp
using Microsoft.AspNetCore.Mvc;
using Newtonsoft.Json;

public class ResultTypesController : Controller
{
    // 1. JsonResult - 處理 JSON 回傳
    public IActionResult GetJsonData()
    {
        var data = new { Name = "張三", Age = 30 };
        return Json(data);
    }

    public IActionResult ProcessJsonResult()
    {
        var result = GetJsonData() as JsonResult;
        
        if (result != null)
        {
            // 取得 JSON 資料
            var jsonData = result.Value;
            
            // 轉換為 JSON 字串
            var jsonString = JsonConvert.SerializeObject(jsonData);
            
            ViewBag.JsonData = jsonString;
            ViewBag.StatusCode = result.StatusCode;
            
            return View();
        }
        
        return View();
    }

    // 2. PartialViewResult - 處理部分檢視
    public IActionResult GetPartialView()
    {
        var model = new List<string> { "項目1", "項目2", "項目3" };
        ViewBag.PartialTitle = "部分檢視標題";
        
        return PartialView("_ItemList", model);
    }

    public IActionResult ProcessPartialResult()
    {
        var result = GetPartialView() as PartialViewResult;
        
        if (result != null)
        {
            // 取得部分檢視的 Model
            var model = result.Model as List<string>;
            
            // 取得部分檢視的 ViewData
            var viewData = result.ViewData;
            
            // 取得部分檢視名稱
            var viewName = result.ViewName;
            
            ViewBag.PartialViewName = viewName;
            ViewBag.PartialModel = model;
            ViewBag.PartialTitle = viewData["PartialTitle"];
            
            return View();
        }
        
        return View();
    }

    // 3. RedirectResult - 處理重新導向
    public IActionResult CreateRedirect()
    {
        return Redirect("/Home/Index");
    }

    public IActionResult ProcessRedirectResult()
    {
        var result = CreateRedirect() as RedirectResult;
        
        if (result != null)
        {
            // 取得重新導向的 URL
            var redirectUrl = result.Url;
            
            // 取得是否為永久重新導向
            var isPermanent = result.Permanent;
            
            ViewBag.RedirectUrl = redirectUrl;
            ViewBag.IsPermanent = isPermanent;
            
            // 根據需要決定是否執行重新導向
            // return result; // 執行重新導向
            return View(); // 或者顯示資訊
        }
        
        return View();
    }

    // 4. RedirectToActionResult - 處理 Action 重新導向
    public IActionResult CreateActionRedirect()
    {
        return RedirectToAction("Index", "Home", new { id = 123 });
    }

    public IActionResult ProcessActionRedirectResult()
    {
        var result = CreateActionRedirect() as RedirectToActionResult;
        
        if (result != null)
        {
            // 取得目標 Action
            var actionName = result.ActionName;
            
            // 取得目標 Controller
            var controllerName = result.ControllerName;
            
            // 取得路由參數
            var routeValues = result.RouteValues;
            
            ViewBag.ActionName = actionName;
            ViewBag.ControllerName = controllerName;
            ViewBag.RouteValues = routeValues;
            
            return View();
        }
        
        return View();
    }

    // 5. FileResult - 處理檔案回傳
    public IActionResult CreateFileResult()
    {
        var fileBytes = System.Text.Encoding.UTF8.GetBytes("這是一個測試檔案");
        return File(fileBytes, "text/plain", "test.txt");
    }

    public IActionResult ProcessFileResult()
    {
        var result = CreateFileResult() as FileResult;
        
        if (result != null)
        {
            // 取得檔案內容類型
            var contentType = result.ContentType;
            
            // 取得檔案名稱（如果有設定）
            var fileName = (result as FileContentResult)?.FileDownloadName;
            
            ViewBag.ContentType = contentType;
            ViewBag.FileName = fileName;
            
            // 如果需要取得檔案內容
            if (result is FileContentResult fileContentResult)
            {
                var fileContent = System.Text.Encoding.UTF8.GetString(fileContentResult.FileContents);
                ViewBag.FileContent = fileContent;
            }
            
            return View();
        }
        
        return View();
    }

    // 6. ContentResult - 處理純文字內容
    public IActionResult CreateContentResult()
    {
        return Content("這是純文字內容", "text/plain", System.Text.Encoding.UTF8);
    }

    public IActionResult ProcessContentResult()
    {
        var result = CreateContentResult() as ContentResult;
        
        if (result != null)
        {
            // 取得內容
            var content = result.Content;
            
            // 取得內容類型
            var contentType = result.ContentType;
            
            ViewBag.Content = content;
            ViewBag.ContentType = contentType;
            
            return View();
        }
        
        return View();
    }

    // 7. StatusCodeResult - 處理狀態碼
    public IActionResult CreateStatusCodeResult()
    {
        return StatusCode(404, "找不到資源");
    }

    public IActionResult ProcessStatusCodeResult()
    {
        var result = CreateStatusCodeResult() as ObjectResult;
        
        if (result != null)
        {
            // 取得狀態碼
            var statusCode = result.StatusCode;
            
            // 取得回應內容
            var value = result.Value;
            
            ViewBag.StatusCode = statusCode;
            ViewBag.ResponseValue = value;
            
            return View();
        }
        
        return View();
    }

    // 8. 通用的 Result 處理方法
    public IActionResult ProcessAnyResult(string resultType)
    {
        IActionResult result = null;
        
        switch (resultType.ToLower())
        {
            case "view":
                result = GetJsonData(); // 這裡可以是任何回傳 ViewResult 的方法
                break;
            case "json":
                result = GetJsonData();
                break;
            case "partial":
                result = GetPartialView();
                break;
            case "redirect":
                result = CreateRedirect();
                break;
            default:
                result = View();
                break;
        }
        
        // 根據 Result 類型進行不同的處理
        ViewBag.ResultType = result.GetType().Name;
        
        switch (result)
        {
            case ViewResult viewResult:
                ViewBag.Model = viewResult.Model;
                ViewBag.ViewData = viewResult.ViewData;
                break;
                
            case JsonResult jsonResult:
                ViewBag.JsonValue = jsonResult.Value;
                break;
                
            case PartialViewResult partialResult:
                ViewBag.PartialModel = partialResult.Model;
                ViewBag.PartialViewName = partialResult.ViewName;
                break;
                
            case RedirectResult redirectResult:
                ViewBag.RedirectUrl = redirectResult.Url;
                break;
        }
        
        return View();
    }
}
```
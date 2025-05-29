---
title: Action Filter
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5, 技術文件]

---

Action Filter 允許我們在控制器的不同階段插入自定義邏輯。

```csharp=
public class CustomActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // 在動作方法執行之前執行
        // 可以用來驗證、修改模型資料或執行前置處理
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        // 在動作方法執行之後執行
        // 可以用來修改結果或執行後置處理
    }

    public override void OnResultExecuting(ResultExecutingContext context)
    {
        // 在結果執行之前執行
        // 可以用來修改視圖資料或調整回應內容
    }

    public override void OnResultExecuted(ResultExecutedContext context)
    {
        // 在結果執行之後執行
        // 可以用來執行清理工作或記錄結果
    }
}
```

```csharp=
[SpmCostKpColDefVMFilter]
public IActionResult CreateConfirmed(SpmCostKpColDefVM model, ReadSpmCostKpCatCondition condition)
{    
    // 當請求進入控制器時，模型綁定器會先處理請求資料並填充模型。
    // 在這個階段，它會根據模型上的驗證特性執行驗證。
    // 如果有任何驗證失敗，這些錯誤就會被記錄在 ModelState 中。
    // 即使我們後續修改了模型的屬性或集合，這些更改也不會影響已經完成的驗證結果。
    // ModelState 是獨立於模型實例的狀態容器。
    // 所以我們先在 ActionFilter 修改了模型的屬性或集合，然後再重新進行驗證
    
    if (!ModelState.IsValid) return View("Create", model);

    model.KpCategory.ModifyUser = User.Identity.Name;
    model.KpCategory.Creator = User.Identity.Name;
    model.KpColumn.ForEach(c =>
    {
        c.ModifyUser = User.Identity.Name;
        c.Creator = User.Identity.Name;
    });   

    TempData["Message"] = "新增完成";
    return RedirectToAction("Create", condition);
}
```

```csharp=
public class SpmCostKpColDefVMFilter: ActionFilterAttribute
{
    /// <summary>
    /// 過濾掉 SpmCostKpColDefVM.KpColumn.IsDelete = true 的資料
    /// 並且重新對 SpmCostKpColDefVM 進行模型驗證
    /// </summary>
    /// <param name="context"></param>
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        // ActionArguments 是一個 Dictionary 型別的資料結構，用於存放所有傳遞給 Action 的參數。
        // 例如當我們在 Action 中定義一個名為 model 的參數時，系統會以 "model" 作為鍵值（Key）儲存在 ActionArguments 中。

        // 我們會使用 TryGetValue 方法來從 Dictionary 中取值，透過 out 參數，方法可以同時完成兩件事
        // (1)回傳布林值，回應是否成功找到對應的值 (2)是若找到值，則將該值存入 modelObj 變數中。
        // 如果找不到 "model" 參數，TryGetValue 會返回 false，而不是拋出異常。
        
        if (context.ActionArguments.TryGetValue("model", out object modelObj))
        {            
            // 檢查 modelObj 是否為 SpmCostKpColDefVM 類型
            // 如果是，則將其轉型並儲存到新的變數 model 中
            if (modelObj is SpmCostKpColDefVM model && model.KpColumn != null)
            {
                // 過濾 SpmCostKpColDefVM.KpColumn.IsDelete = true
                model.KpColumn = model.KpColumn.Where(c => !c.IsDelete).ToList();
                                
                // 清除之前的驗證結果
                context.ModelState.Clear();
                
                // 重新對 SpmCostKpColDefVM 進行模型驗證
                var controller = context.Controller as Controller;
                controller.TryValidateModel(model);
                
                // TryValidateModel 的行為是「只有在驗證失敗時才會把結果加入 ModelState」。
                // ★ 如果驗證成功，它不會在 ModelState 中新增任何紀錄。
                // 驗證成功回傳 True，失敗回傳 False
            }
        }
        
        base.OnActionExecuting(context);
    }
    
    public override void OnActionExecuted(ActionExecutedContext context)
    {
        // 如果需要在動作執行後進行處理，可以在這裡添加邏輯
        if (context.Exception != null)
        {
            // 處理執行過程中發生的異常
        }

        base.OnActionExecuted(context);
    }    
}
```

```csharp=
public class SpmCostKpColDefVM
{
    public SpmCostKpCatVM KpCategory { get; set; }
    public List<SpmCostKpColVM> KpColumn { get; set; }
}

public class SpmCostKpCatVM
{
    [Display(Name = "ID")]
    public int Id { get; set; }

    [Required(ErrorMessage = "請輸入Category Name")]
    [RegularExpression(@".*\S+.*", ErrorMessage = "請勿只輸入空白字元")]
    [StringLength(255, ErrorMessage = "請輸入255字元以內")]
    [Display(Name = "Category Name")]        
    public string Name { get; set; }

    [Display(Name = "Mkt Spec")]
    public int? MktSpecId { get; set; }

    [Display(Name = "維護人員")]
    public string ModifyUser { get; set; }

    [Display(Name = "維護時間")]
    public DateTime ModifyDate { get; set; }

    [Display(Name = "建立人員")]
    public string Creator { get; set; }

    [Display(Name = "建立時間")]
    public DateTime CreateDate { get; set; }
}

public class SpmCostKpColVM
{
    [Display(Name = "ID")]
    public int Id { get; set; }

    [Required(ErrorMessage = "請輸入欄位名稱")]
    [RegularExpression(@".*\S+.*", ErrorMessage = "請勿只輸入空白字元")]
    [StringLength(255, ErrorMessage = "請輸入255字元以內")]
    [Display(Name = "欄位名稱")]        
    public string Name { get; set; }

    [Required(ErrorMessage = "請輸入Category ID")]
    [Display(Name = "Category ID")]
    public int KPCategoryId { get; set; }

    [Required(ErrorMessage = "請輸入排序值")]
    [Display(Name = "排序")]
    [Range(0, int.MaxValue, ErrorMessage = "排序值必須大於或等於0")]
    public int Sequence { get; set; }

    [Display(Name = "維護人員")]
    public string ModifyUser { get; set; }

    [Display(Name = "維護時間")]
    public DateTime ModifyDate { get; set; }

    [Display(Name = "建立人員")]
    public string Creator { get; set; }

    [Display(Name = "建立時間")]
    public DateTime CreateDate { get; set; }

    [Display(Name="前端刪除標記")]
    public bool IsDelete { get; set; }
}
```
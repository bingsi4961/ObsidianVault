---
title: ModelState
tags: [ASP.NET Core MVC 3.1]

---

首先，讓我們從最上層的 ModelState 開始說明其結構：

```csharp
/// <summary>
/// ModelStateDictionary 類別是 ASP.NET MVC 中用於追蹤整個模型驗證狀態的核心類別
/// 它實作了 IEnumerable<KeyValuePair<string, ModelStateEntry>>，允許我們遍歷所有驗證項目
/// </summary>
public class ModelStateDictionary : IEnumerable<KeyValuePair<string, ModelStateEntry>> 
{
    /// <summary>
    /// 表示整個模型的驗證狀態
    /// 當所有屬性都通過驗證時為 true，只要有任何一個驗證失敗就為 false
    /// 這是最常用來檢查表單提交是否有效的屬性
    /// </summary>
    public bool IsValid { get; }

    /// <summary>
    /// 包含所有驗證項目的鍵值集合
    /// 鍵通常對應到模型中的屬性名稱，例如 "UserName"、"Email" 等
    /// 可用於遍歷所有需要驗證的欄位
    /// </summary>
    public IEnumerable<string> Keys { get; }

    /// <summary>
    /// 包含所有模型狀態項目的集合
    /// 每個 ModelStateEntry 物件包含該屬性的驗證狀態、錯誤訊息等詳細資訊
    /// 用於取得所有屬性的驗證結果
    /// </summary>
    public IEnumerable<ModelStateEntry> Values { get; }

    /// <summary>
    /// 代表模型最頂層的驗證項目
    /// 用於存放不特定於任何屬性的模型級別驗證錯誤
    /// 例如，密碼與確認密碼不相符這類跨欄位的驗證錯誤
    /// </summary>
    public ModelStateEntry Root { get; }

    /// <summary>
    /// 統計整個模型中所有驗證錯誤的總數
    /// 包含所有屬性的錯誤數量總和
    /// 可用於快速判斷是否有任何驗證錯誤，以及錯誤的嚴重程度
    /// </summary>
    public int ErrorCount { get; }
    
    /// <summary>
    /// 新增一個模型驗證錯誤
    /// </summary>
    /// <param name="key">發生錯誤的屬性名稱</param>
    /// <param name="errorMessage">錯誤訊息描述</param>
    public void AddModelError(string key, string errorMessage)
    {
        // 在指定的 key 下添加新的錯誤信息
        // 如果 key 不存在，會自動創建新的 ModelStateEntry
    }

    /// <summary>
    /// 清除所有模型狀態資訊
    /// 包含驗證錯誤、屬性值等所有相關資訊
    /// </summary>
    public void Clear()
    {
        // 重置所有驗證狀態和錯誤信息
    }

    /// <summary>
    /// 檢查指定的屬性名稱是否存在於模型狀態字典中
    /// </summary>
    /// <param name="key">要檢查的屬性名稱</param>
    /// <returns>如果屬性存在則回傳 true，否則回傳 false</returns>
    public bool ContainsKey(string key)
    {
        // 返回特定 key 是否存在於字典中
    }

    /// <summary>
    /// 從模型狀態字典中移除指定屬性的狀態資訊
    /// ★★ 會同時刪除「使用者輸入的值、驗證結果」
    /// </summary>
    /// <param name="key">要移除的屬性名稱</param>
    /// <returns>如果成功移除則回傳 true，如果屬性不存在則回傳 false</returns>
    public bool Remove(string key)
    {
        // 從字典中移除指定 key 的所有驗證信息
    }

    /// <summary>
    /// 嘗試取得指定屬性的模型狀態資訊
    /// </summary>
    /// <param name="key">要查詢的屬性名稱</param>
    /// <param name="value">如果屬性存在，則透過此參數回傳對應的模型狀態資訊</param>
    /// <returns>如果找到指定的屬性則回傳 true，否則回傳 false</returns>
    public bool TryGetValue(string key, out ModelStateEntry value)
    {
        // 安全地取得指定 key 的驗證信息
    }
    
    /// <summary>
    /// 透過索引器取得或設定指定屬性的模型狀態
    /// </summary>
    /// <param name="key">屬性名稱</param>
    /// <returns>對應的 ModelStateEntry 物件，如果不存在則回傳 null</returns>
    public ModelStateEntry this[string key]
    {
        get
        {
            ModelStateEntry entry;
            TryGetValue(key, out entry);
            return entry;
        }
    }
}

/// <summary>
/// ModelStateEntry 類別代表單一屬性的驗證狀態
/// 包含該屬性的輸入值、驗證結果和錯誤訊息等完整資訊
/// </summary>
public class ModelStateEntry 
{
    /// <summary>
    /// 使用者實際輸入的原始字串值
    /// 例如，如果使用者在日期欄位輸入了 "2024-01-01"，這個值就會被保存在這裡
    /// 即使轉換失敗，也可以保留使用者的原始輸入用於顯示
    /// </summary>
    public string AttemptedValue { get; set; }

    /// <summary>
    /// 模型綁定嘗試轉換後的原始值
    /// 例如，若 AttemptedValue 為 "2024-01-01"，這裡可能會是對應的 DateTime 物件
    /// 如果轉換失敗，可能為 null 或保持原始型態
    /// </summary>
    public object RawValue { get; set; }

    /// <summary>
    /// 該屬性的所有驗證錯誤集合
    /// 一個屬性可能同時違反多個驗證規則，每個錯誤都會加入這個集合
    /// 例如，密碼可能同時違反長度和複雜度要求
    /// </summary>
    public ModelErrorCollection(Collection<ModelError>) Errors { get; }

    /// <summary>
    /// 表示該屬性目前的驗證狀態
    /// 使用 ModelValidationState 列舉值來標示是否已驗證、驗證結果等
    /// 這個狀態會影響 ModelStateDictionary.IsValid 的結果
    /// </summary>
    public ModelValidationState ValidationState { get; set; }
}

/// <summary>
/// ModelError 類別用於表示單一個驗證錯誤
/// 包含錯誤訊息和可能的例外狀況詳細資訊
/// </summary>
public class ModelError 
{
    /// <summary>
    /// 人類可讀的錯誤訊息
    /// 通常用於在視圖中顯示給使用者看的驗證失敗說明
    /// 例如 "密碼長度必須至少為 6 個字元" 等提示訊息
    /// </summary>
    public string ErrorMessage { get; }

    /// <summary>
    /// 造成驗證失敗的例外狀況物件
    /// 可能包含更詳細的技術資訊，有助於偵錯
    /// 例如型別轉換失敗的詳細原因
    /// </summary>
    public Exception Exception { get; }
}

/// <summary>
/// ModelValidationState 列舉定義了屬性可能的驗證狀態
/// 用於追蹤驗證過程的結果和進度
/// </summary>
public enum ModelValidationState 
{
    /// <summary>
    /// 該屬性尚未進行任何驗證
    /// 這是屬性的初始狀態
    /// </summary>
    Unvalidated = 0,

    /// <summary>
    /// 該屬性已驗證但未通過驗證規則
    /// 表示存在驗證錯誤
    /// </summary>
    Invalid = 1,

    /// <summary>
    /// 該屬性已通過所有驗證規則
    /// 表示值符合所有要求
    /// </summary>
    Valid = 2,

    /// <summary>
    /// 該屬性的驗證被刻意跳過
    /// 通常用於條件性驗證，某些情況下不需要驗證的欄位
    /// </summary>
    Skipped = 3
}
```

這些類別共同組成了 ASP.NET MVC 的模型驗證框架。當我們提交一個表單時：

1. 框架會建立一個 `ModelStateDictionary` 實例來追蹤整個驗證過程
2. 對於表單中的每個欄位，會建立對應的 `ModelStateEntry` 來記錄該欄位的驗證狀態
3. 如果驗證失敗，會建立 `ModelError` 物件來記錄具體的錯誤資訊
4. 使用 `ModelValidationState` 列舉來標示每個欄位的驗證結果

舉個例子，假設我們有一個使用者註冊的表單，包含用戶名和密碼兩個欄位。當表單提交時：

1. 框架會建立一個 ModelStateDictionary 實例
2. 如果用戶名為空，可能會呼叫 AddModelError("Username", "用戶名不能為空")
3. 如果密碼太短，可能會呼叫 AddModelError("Password", "密碼至少需要 6 個字元")
4. 之後可以透過 IsValid 屬性來檢查整個模型是否有效
5. 如果需要顯示錯誤訊息，可以使用 Values 屬性來取得所有的驗證錯誤

---
讓我用一個實際的例子來說明這些屬性是如何協同工作的：

```csharp
public class UserRegistration
{
    [Required]
    [StringLength(50)]
    public string Username { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }
}

// 控制器中的使用示例
public IActionResult Register(UserRegistration model)
{
    if (!ModelState.IsValid)  // 檢查整體驗證狀態
    {
        foreach (var modelStateEntry in ModelState.Values)
        {
            if (modelStateEntry.ValidationState == ModelValidationState.Invalid)
            {
                // 獲取嘗試輸入的值
                string attemptedValue = modelStateEntry.AttemptedValue;
                
                // 獲取驗證錯誤訊息
                foreach (var error in modelStateEntry.Errors)
                {
                    string errorMessage = error.ErrorMessage;
                    // 處理錯誤...
                }
            }
        }
    }
}
```

```csharp
public class RegisterViewModel
{
    [Required(ErrorMessage = "用戶名稱是必填的")]
    public string Username { get; set; }

    [Required(ErrorMessage = "電子郵件是必填的")]
    [EmailAddress(ErrorMessage = "請輸入有效的電子郵件地址")]
    public string Email { get; set; }
}

public class AccountController : Controller
{
    public IActionResult Register(RegisterViewModel model)
    {
        // 檢查整體模型驗證狀態
        if (!ModelState.IsValid)
        {
            // 獲取錯誤數量
            var totalErrors = ModelState.ErrorCount;

            // 遍歷所有驗證條目
            foreach (var key in ModelState.Keys)
            {
                // 嘗試獲取特定欄位的驗證狀態
                if (ModelState.TryGetValue(key, out ModelStateEntry entry))
                {
                    if (entry.ValidationState == ModelValidationState.Invalid)
                    {
                        // 處理錯誤
                        foreach (var error in entry.Errors)
                        {
                            // 記錄或處理錯誤訊息
                            var errorMessage = error.ErrorMessage;
                        }
                    }
                }
            }

            // 手動添加自訂錯誤
            ModelState.AddModelError("CustomError", "發生自訂錯誤");

            // 清除特定欄位的驗證狀態
            ModelState.Remove("Username");

            // 完全清除所有驗證狀態
            ModelState.Clear();
        }

        return View(model);
    }
}
```

```csharp
public class UserRegistration
{
    [Required(ErrorMessage = "用戶名稱是必填的")]
    [StringLength(50, MinimumLength = 3, ErrorMessage = "用戶名稱必須介於 3 到 50 個字元之間")]
    public string Username { get; set; }

    [Required(ErrorMessage = "電子郵件是必填的")]
    [EmailAddress(ErrorMessage = "請輸入有效的電子郵件地址")]
    public string Email { get; set; }
}

public class AccountController : Controller
{
    public IActionResult Register(UserRegistration model)
    {
        // 檢查模型驗證狀態
        if (!ModelState.IsValid)
        {
            // 取得特定欄位的驗證狀態
            if (ModelState.TryGetValue("Username", out var usernameEntry))
            {
                // 處理該欄位的所有錯誤
                foreach (var error in usernameEntry.Errors)
                {
                    // 取得錯誤訊息
                    string message = error.ErrorMessage;
                    
                    // 如果有相關的異常
                    if (error.Exception != null)
                    {
                        // 記錄異常詳細資訊
                        Logger.LogError(error.Exception);
                    }
                }

                // 展示如何使用 ModelErrorCollection 的各種方法
                var errorCollection = usernameEntry.Errors;

                // 取得錯誤數量
                int errorCount = errorCollection.Count;

                // 添加新的錯誤訊息
                errorCollection.Add("自訂錯誤訊息");

                // 添加異常
                try
                {
                    // 某些可能拋出異常的操作
                    throw new ArgumentException("無效的用戶名稱格式");
                }
                catch (Exception ex)
                {
                    errorCollection.Add(ex);
                }

                // 清除所有錯誤
                errorCollection.Clear();
            }
        }

        return View(model);
    }
}
```

讓我們來看看如何使用這個索引器的實際例子：

```csharp
public class RegisterController : Controller
{
    public IActionResult Register(RegisterViewModel model)
    {
        // 假設我們在驗證使用者註冊表單

        // 1. 檢查使用者名稱
        if (string.IsNullOrEmpty(model.Username))
        {
            ModelState.AddModelError("Username", "使用者名稱不能為空");
        }

        // 2. 檢查密碼
        if (model.Password?.Length < 6)
        {
            ModelState.AddModelError("Password", "密碼至少需要6個字元");
        }

        // 3. 使用索引器來取得特定欄位的驗證狀態
        ModelStateEntry usernameState = ModelState["Username"];
        if (usernameState != null && usernameState.Errors.Any())
        {
            // 取得使用者名稱欄位的第一個錯誤訊息
            string firstError = usernameState.Errors[0].ErrorMessage;
            // 這裡的 firstError 就會是 "使用者名稱不能為空"
            
            // 可以進行額外的處理...
        }

        // 4. 檢查密碼欄位的驗證狀態
        var passwordState = ModelState["Password"];
        if (passwordState?.Errors.Count > 0)
        {
            // 取得所有密碼相關的錯誤訊息
            foreach (var error in passwordState.Errors)
            {
                string errorMessage = error.ErrorMessage;
                // 進行錯誤處理...
            }
        }

        if (!ModelState.IsValid)
        {
            return View(model);
        }

        // 繼續處理有效的註冊請求...
        return RedirectToAction("Success");
    }
}
```
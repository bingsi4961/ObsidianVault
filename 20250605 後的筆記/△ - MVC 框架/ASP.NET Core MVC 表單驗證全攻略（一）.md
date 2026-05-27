---
date: 2026-05-24 22:12
title:
aliases:
tags:
  - Validation
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[jQuery Validation 三個檔案說明]]
#### 📑 [[ASP.NET Core MVC 表單驗證全攻略（二）]]
#### 📑 [[ASP.NET Core MVC 表單驗證全攻略（三）]]

---
## 一、為什麼需要這套機制？先看問題本身

在你還沒接觸這套驗證機制之前，你一定寫過這樣的程式碼：前端用一堆 JavaScript `if/else` 檢查使用者的輸入，然後資料送到後端，你又在 Controller 裡面寫一次差不多的 `if/else`。兩套邏輯放在完全不同的地方，規則一改就要改兩份，非常難維護。

ASP.NET Core 提供的「**模型驗證（Model Validation）**」機制，目的就是解決這個問題：**只要在 C# 的 Model 裡面把規則定義好，前端和後端的驗證就可以同時生效，完全不需要重複造輪子**。

整套機制有三個明確的角色分工，你只要記住這個大原則，後面所有的細節都會變得很好理解：

- **Model（資料模型）**：負責定義「規則」。用標籤（Annotation）把安檢條件貼在每個屬性上。
- **View（畫面）**：負責「顯示」錯誤給使用者看。告訴瀏覽器錯誤要顯示在哪裡。
- **Controller（控制器）**：負責「最後把關」。後端的終極防線，確保資料安全無誤。

---
## 二、整體架構比喻：機場海關安檢 🛂

在深入程式碼之前，我想給你一個具體的畫面感，方便你在腦中建立整套機制的地圖。

想像機場的安檢流程：

**Data Annotation（資料註解）** 就像是「海關的違禁品清單」。這份清單貼在每件行李（Model 屬性）身上，寫明了「液體不能超過 100ml」、「必須攜帶護照」這種死規則。

**`asp-validation-for`** 就像是「安檢人員指著你的特定背包說：先生，你這個背包裡面有剪刀」。它放在每個輸入框旁邊，針對「單一欄位」給出精準的錯誤提示。

**`asp-validation-summary`** 就像是「機場大廳的大螢幕廣播」。它會一次把所有出問題的地方列出來，或是用來顯示跟特定背包無關的整體性錯誤（例如：這班飛機停飛了）。

這三個角色各司其職，接下來我們一個個拆解。

---
## 三、第一塊拼圖：Model Annotation（在屬性上貼標籤）

### 基本觀念

Annotation 的作用就是在 Model 的屬性頭上「貼標籤」，告訴系統這個屬性的規則是什麼。不要死背，把它想成你在為資料貼上說明書。

```csharp
// RegisterViewModel.cs
using System.ComponentModel.DataAnnotations;

public class RegisterViewModel
{
    // [Required] 確保必填，沒填就跳出 ErrorMessage
    [Required(ErrorMessage = "姓名是必填欄位")]
    public string Name { get; set; }

    // [EmailAddress] 自動幫你檢查是不是合法的 Email 格式
    [Required(ErrorMessage = "信箱不能為空")]
    [EmailAddress(ErrorMessage = "信箱格式不正確")]
    public string Email { get; set; }
}
```

> 💡 **學長提醒**：注意每個 Annotation 上面的 `ErrorMessage` 參數——這就是當驗證失敗時，畫面上要顯示給使用者的訊息。如果你不填，系統會套用預設的英文訊息，在中文系統裡看起來非常突兀，記得養成習慣都加上去。

---
### 常用的內建 Annotation 一覽

以下是你在日常 CRUD 開發中最常用到的內建標籤，搭配 `ErrorMessage` 的完整寫法：

| C# Annotation                                              | 用途說明                 |
| ---------------------------------------------------------- | -------------------- |
| `[Required(ErrorMessage="必填")]`                            | 確保欄位不為 null 或空字串     |
| `[EmailAddress(ErrorMessage="格式錯誤")]`                      | 驗證 Email 格式（含 @ 與網域） |
| `[StringLength(10, ErrorMessage="不得超過10個字元")]`             | 限制字串最大長度             |
| `[StringLength(10, MinimumLength=3, ErrorMessage="長度不對")]` | 同時限制最大與最小長度          |
| `[Range(1, 100, ErrorMessage="超出範圍")]`                     | 限制數值必須在指定範圍內         |
| `[RegularExpression(@"^\d+$", ErrorMessage="限數字")]`        | 用正規表達式自訂格式           |
| `[Compare("Password", ErrorMessage="密碼不一致")]`              | 比對另一個欄位的值（最常用在確認密碼）  |
| `[Phone(ErrorMessage="電話格式錯誤")]`                           | 驗證電話號碼格式             |

---
### 什麼時候「不該」用 Annotation？

Annotation 很好用，但不要把所有驗證邏輯都塞進去。有兩個重要邊界你要記住：

**✅ 適合用 Annotation 的情境**：單純的格式檢查，例如字串長度、必填、數字範圍、Email 格式。這些規則是「死的」，不論在哪個情境都不會改變。

**❌ 不適合用 Annotation 的情境**：涉及資料庫查詢或複雜業務邏輯，例如「這個帳號是否已被其他人註冊」、「如果會員等級是 VIP，折扣碼才是必填」。

原因很簡單：**Model 應該是單純的資料載體，不應該去呼叫資料庫**，也不適合寫一堆 `if/else` 判斷。這類複雜邏輯應該留在 Controller 或 Service 層裡面處理，搭配我們後面會講到的 `ModelState.AddModelError()` 來發送錯誤。

> 💡 **用一句話記住**：Annotation 只負責「基本安檢（有沒有帶護照）」，複雜的「身家調查（這本護照有沒有被通緝）」請交給後端的商業邏輯來處理。

---
## 四、第二塊拼圖：View 裡的 Tag Helper（顯示錯誤訊息）

### asp-validation-for 與 asp-validation-summary 的分工

知道規則怎麼定義了，接下來要讓錯誤顯示在畫面上。在 .NET Core 裡，微軟推廣使用 **Tag Helper**（也就是 `asp-*` 開頭的屬性），它看起來就像原生 HTML，對前端開發者非常友善。

```html
<form asp-action="Register" method="post">

    <!-- 大螢幕廣播：顯示整體性的錯誤（ModelOnly 模式） -->
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
        <!-- 單一欄位的錯誤提示：有錯就顯示在輸入框旁邊 -->
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Email"></label>
        <input asp-for="Email" class="form-control" />
        <span asp-validation-for="Email" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">註冊</button>
</form>
```

---
### asp-validation-summary 的三種模式，到底差在哪裡？

`asp-validation-summary` 有三種設定值，很多人搞不清楚差異，我們用一個實際情境來說明。

假設使用者送出表單後，系統抓到了兩個錯誤：

1. **密碼欄位沒填**（這是針對特定欄位的錯誤）
2. **帳號與密碼不相符**（這是整體性錯誤，因為單看帳號或密碼都沒錯，合在一起才錯）

|模式|大螢幕上會顯示什麼|實務建議|
|---|---|---|
|`All`|顯示所有錯誤，包含欄位錯誤和整體錯誤|當你的表單沒有在每個輸入框旁放 `asp-validation-for` 時使用；缺點是若兩者同時存在，錯誤會在畫面上重複出現|
|`ModelOnly`|**只顯示整體性錯誤**，不顯示綁定在特定欄位的錯誤|最推薦、最常用的模式。由 `asp-validation-for` 負責欄位錯誤，上方大螢幕只負責系統層級的訊息，版面最乾淨|
|`None`|什麼都不顯示|幾乎不用，除非你要自己用 JavaScript 完全接手錯誤訊息的渲染|

> 💡 **最佳實務**：在畫面頂部放一個 `ModelOnly` 的大螢幕，然後每個輸入框旁邊各放一個 `asp-validation-for`，這是分工最清晰的標準寫法。

---
## 五、第三塊拼圖：Controller 的最後把關

### 基本流程

Controller 的工作是在資料送到後端時做最終確認。標準寫法如下：

```csharp
[HttpPost]
public IActionResult Register(RegisterViewModel model)
{
    // 最常見的第一行：IsValid 會幫你確認所有 Annotation 規則都通過了
    if (!ModelState.IsValid)
    {
        // 記得把 model 傳回去，使用者填寫的內容才不會消失！
        return View(model);
    }

    // 驗證通過，繼續處理業務邏輯（例如存入資料庫）
    // ...
    return RedirectToAction("Success");
}
```

> ⚠️ **新手地雷**：很多人第一次寫會寫 `return View()` 而不是 `return View(model)`。差別在於：`model` 是使用者剛才填寫的表單資料，如果你不把它傳回去，網頁重新載入後，使用者辛苦填寫的所有輸入框都會變成空白！`ModelState` 雖然會自動帶回前端（錯誤訊息還在），但輸入框的值需要靠 `model` 來還原。

---
### 手動把錯誤塞進 ModelState

如果是需要查資料庫才能判斷的業務邏輯錯誤（例如帳號重複），你需要用 `ModelState.AddModelError()` 手動把錯誤塞進去：

```csharp
[HttpPost]
public IActionResult Register(RegisterViewModel model)
{
    if (!ModelState.IsValid)
    {
        return View(model);
    }

    // 去資料庫查帳號是否重複
    if (CheckFromDatabase(model.Account))
    {
        // 把錯誤綁定到特定欄位：帳號輸入框旁邊會出現紅字
        ModelState.AddModelError("Account", "這個帳號名稱已被註冊");
        return View(model);
    }

    // ...
}
```

第一個參數（`key`）的選擇非常重要，有兩種用法：

**指定欄位名稱**（例如 `"Account"`）：錯誤會精準地顯示在對應輸入框旁邊的 `asp-validation-for` 標籤。這是大多數情況下更好的選擇，使用者一眼就知道哪個欄位填錯了。

**空字串**（`string.Empty` 或 `""`）：錯誤會被視為整體性錯誤，由畫面頂部的 `asp-validation-summary="ModelOnly"` 的大螢幕顯示。適合用在「帳號或密碼錯誤」這種為了安全性故意不明說是哪個欄位錯了的情境。

```csharp
// 整體性錯誤，顯示在 ModelOnly 大螢幕
ModelState.AddModelError(string.Empty, "帳號或密碼輸入錯誤，請重新輸入。");

// 特定欄位錯誤，顯示在 Email 輸入框旁邊
ModelState.AddModelError("Email", "此信箱已被其他人註冊");
```

---
## 六、雙重防護網：為什麼前後端都要檢查？

這是一個很多新手會問的問題：既然後端 C# 都會驗證了，前端還需要驗證嗎？

**絕對需要，而且缺一不可。** 兩道防線各有不同的目的：

**第一道防線（前端 / 瀏覽器）**：反應快，使用者一填錯就立即提示，不需要等到整個網頁重新整理，大幅提升使用者體驗，同時也節省伺服器資源。

**第二道防線（後端 / 伺服器）**：最終把關，絕對安全。因為前端的 JavaScript 防護可能被使用者手動關閉，或是駭客直接用工具（如 Postman、curl）繞過網頁發送請求，這時後端是唯一的屏障。

> 💡 **前端防護君子，後端防護小人。** 永遠不要相信前端傳來的資料。

---
### 神奇之處：Annotation 同時生效於前後端

這套機制最聰明的地方在於：**你只要在 C# 裡寫一次 `[Required]`，前後端的驗證都會自動生效**。

當 ASP.NET Core 把畫面渲染成 HTML 送給瀏覽器時，它會把 C# 的 Annotation「翻譯」成 HTML 標籤上的 `data-val-*` 屬性。例如 `[Required]` 會變成 `data-val-required="此欄位必填"`。

接著，微軟預先幫你整合了一個叫做 **jQuery Unobtrusive Validation** 的 JavaScript 套件。這個套件會去尋找畫面上所有帶有 `data-val-*` 屬性的輸入框，在使用者按下送出的瞬間，在前端做第一道攔截。

如果使用者關掉了 JavaScript，前端防線失效，表單被送到後端，這時 C# 的 `[Required]` 規則就會再次生效，把錯誤記錄到 `ModelState` 裡。

整個流程的生命週期是：**C# Annotation 定規則 → 翻譯成 HTML `data-val-*` 屬性 → jQuery 讀取屬性做前端攔截 → 後端 ModelState 做最終把關**。

---
## 七、別忘了載入前端防護腳本！

有了以上這些，你還差最後一塊拼圖：**前端的 jQuery Unobtrusive Validation 腳本必須主動載入到網頁中**，品管員才會站崗。

在 ASP.NET Core MVC 專案裡，微軟已經把這段腳本打包成一個 Partial View：`_ValidationScriptsPartial.cshtml`。你需要在每個使用驗證的 View 最底部加上：

```csharp
@section Scripts {
    @{
        await Html.RenderPartialAsync("_ValidationScriptsPartial");
    }
}
```

> 💡 **為什麼要放在 `@section Scripts`？** 這個 section 對應到全站母版（`_Layout.cshtml`）的 HTML 最底部。把 JavaScript 放在最底部是業界慣例，可以避免腳本載入時阻塞畫面的渲染，讓網頁顯示更快。

---
## 八、完整流程總覽

到這裡，我們已經把整套機制的拼圖拼完了。用一張表格幫你做個總結：

| 角色                           | 位置               | 負責的事                    |
| ---------------------------- | ---------------- | ----------------------- |
| `[Required]` 等 Annotation    | `ViewModel.cs`   | 定義安檢規則，會自動翻譯成前後端都認識的形式  |
| `asp-validation-for`         | `.cshtml` (View) | 在單一輸入框旁邊顯示該欄位的錯誤訊息      |
| `asp-validation-summary`     | `.cshtml` (View) | 在表單頂部顯示整體性錯誤訊息          |
| `_ValidationScriptsPartial`  | `.cshtml` (View) | 載入前端 JS 品管員，讓前端攔截機制正式生效 |
| `ModelState.IsValid`         | `Controller.cs`  | 後端最終把關，確認所有規則都通過        |
| `ModelState.AddModelError()` | `Controller.cs`  | 手動把複雜業務邏輯的錯誤塞入驗證系統      |

---
## 延伸：遠端驗證（`[Remote]`）—— 連通前後端的即時橋樑

有一種特殊的 Annotation 值得特別介紹：`[Remote]`。它解決的問題是「有些事前端不可能自己知道（例如這個帳號是否已被其他人用了）」，以前只能等整張表單送出才知道。

`[Remote]` 讓前端可以在使用者填完欄位離開後，偷偷發一個 AJAX 請求去問後端，把結果即時顯示出來，不需要整頁重新整理：

```csharp
// Model 端：指定去呼叫哪個 Controller 的哪個 Action 做驗證
public class RegisterViewModel
{
	// 這是你要一起帶過去的第二個參數
	public int CompanyId { get; set; }
    
    // 這是你要一起帶過去的第三個參數 
    public int DepartmentId { get; set; }
    
    [Required(ErrorMessage = "帳號必填")]
    [Remote(action: "CheckAccountDuplicate", controller: "User", 
	    AdditionalFields = nameof(CompanyId) + "," + nameof(DepartmentId), ErrorMessage = "這個帳號已經被註冊囉")]
    public string Account { get; set; }
}
```

```csharp
// Controller 端：需要兩個 Action
public class UserController : Controller
{
    // ❶ 給前端 AJAX 呼叫的「查哨站」—— 只能回傳 JSON
    [AcceptVerbs("GET", "POST")]
    public IActionResult CheckAccountDuplicate(string account, int companyId, int departmentId)
    {
        bool isExist = CheckFromDatabase(account, companyId, departmentId);
        if (isExist)
        {
            // 回傳字串 = 驗證失敗，字串內容就是錯誤訊息
            return Json($"帳號 {account} 已經有人用囉！");
        }

        // 回傳 true = 驗證通過
        return Json(true);
    }

    // ❷ 最終接收整張表單的「真實防線」
    [HttpPost]
    public IActionResult Register(RegisterViewModel model)
    {
        // 🚨 重要！[Remote] 只管前端體驗，後端不會自動執行 Remote 的檢查
        // 你必須在這裡手動再查一次資料庫！
        if (CheckFromDatabase(model.Account, model.CompanyId, model.DepartmentId))
        {
            ModelState.AddModelError("Account", "這個帳號已經被註冊囉");
        }

        if (!ModelState.IsValid)
        {
            return View(model);
        }

        // 儲存進資料庫...
        return RedirectToAction("Success");
    }
}
```

> ⚠️ **`[Remote]` 最大的地雷**：`[Remote]` 的設計初衷「純粹是為了前端的體驗（UX）」。當表單整包送到後端、執行 `ModelState.IsValid` 時，系統不會自動再去呼叫 Remote 的查哨站。因此，你必須在接收表單的 Action 裡**手動再查一次資料庫**，並用 `ModelState.AddModelError()` 把錯誤加入。 `[Remote]` 只防君子，不防小人。
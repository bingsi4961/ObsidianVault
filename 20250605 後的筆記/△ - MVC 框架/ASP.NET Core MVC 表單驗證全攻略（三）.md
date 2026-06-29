---
date: 2026-05-25 13:52
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
#### 📑 [[ASP.NET Core MVC 表單驗證全攻略（一）]]
#### 📑 [[ASP.NET Core MVC 表單驗證全攻略（二）]]

---
## 一、C# 和瀏覽器的語言障礙：誰來翻譯？

瀏覽器只看得懂 HTML、CSS 和 JavaScript，完全不認識 C# 或 `[Required]` 這種標籤。但我們不想在前端把所有驗證規則用 JavaScript 重寫一遍 -- 這樣會違背「避免重複造輪子」的精神，後端規則改了，前端還要記得跟著改，遲早會漏改、出 bug。

微軟的解法是利用 **HTML5 的自訂資料屬性（`data-*`）** 作為溝通橋樑。當 ASP.NET Core 把畫面渲染成 HTML 時，它會把 C# 的 Annotation 「翻譯」成 HTML 標籤上的 `data-val-*` 屬性，讓前端的 JavaScript 可以讀懂。

---
## 二、前端驗證的兩個角色：品管員 vs 翻譯橋樑

在搞懂這些 HTML 屬性之前，你需要先認識兩個角色的分工，否則你會一直搞混哪些東西是誰負責的。

**jQuery Validation（核心品管員）**：這是第三方開源套件 [jQuery Validation Plugin](https://jqueryvalidation.org/) (`jquery.validate.js`)，它是前端驗證的核心引擎，只聽得懂 JavaScript 語言。如果你直接用它，需要寫一段 JS 程式碼手動設定規則。

**jQuery Unobtrusive Validation（微軟翻譯橋樑）**：微軟專為 ASP.NET 開發的配接器套件（`jquery.validate.unobtrusive.js`）。它的工作是在網頁載入時，讀取畫面上所有的 `data-val-*` 屬性，自動翻譯成核心品管員聽得懂的規則，並登記到品管員的規則本裡。**你不需要手動設定任何 JS 規則，C# 的 Annotation 就會自動生效。**

簡單說：**MVC 的 Annotation 機制靠的是「翻譯橋樑」，不是純 jQuery 驗證**。你在公司系統裡看到那些 `data-val-*` 屬性，就是翻譯橋樑的傑作。

> 💡 **換個說法你可能也會聽到**：有些資料會把 `jquery.validate.js` 稱為「執行引擎」，把 `jquery.validate.unobtrusive.js` 稱為「翻譯官」。這其實跟「品管員」和「翻譯橋樑」是同一套分工，只是比喻用詞不同。日後你查資料、問別人的時候，如果聽到不一樣的稱呼，先確認一下講的是不是同一個東西，不要被名詞絆住。

---
## 三、從 C# 到瀏覽器：完整運作流程實境拆解

光講「兩個角色」還是有點抽象，這一章我們把整個流程拆成四個步驟，實際走一遍，你就會徹底搞懂這兩支 JS 檔案是怎麼合作的。

### 步驟一：後端 C# 定義規則（Data Annotation）

你在 Model 加上驗證屬性：

```csharp
public class UserModel
{
    [Required(ErrorMessage = "請輸入姓名")]
    public string Name { get; set; }
}
```

### 步驟二：Razor 產生帶有 data-val 的 HTML

不管你是用 .NET Framework 的 `@Html.TextBoxFor(m => m.Name)`，還是 .NET Core 的 `<input asp-for="Name" />`，MVC 框架都會自動把 C# 的屬性轉換成 HTML5 的 `data-val-*` 屬性：

```html
<input type="text" id="Name" name="Name"
       data-val="true"
       data-val-required="請輸入姓名" />
```

> 【補充說明】這一步剛好對你目前的工作很實用：Portal 用的是 .NET Core 3.1（`asp-for` Tag Helper），GTS 用的是 .NET Framework 4.8（`Html.TextBoxFor`），但兩者底層產生 `data-val-*` 屬性的機制是相通的。也就是說，這整篇文章接下來講的所有驗證原理，**在你的兩套系統裡都適用**，不需要因為框架版本不同而另外學一套。

（注意：到這一步為止，純粹只是 HTML 產出，還沒有任何 JavaScript 驗證效果。）

### 步驟三：網頁載入，翻譯橋樑開始工作（翻譯）

當網頁載入完成後，`jquery.validate.unobtrusive.js`（翻譯橋樑）會去尋找畫面上所有帶有 `data-val="true"` 的元素。它看到 `data-val-required="請輸入姓名"` 後，會在背景動態組裝出一份品管員看得懂的設定檔，類似這樣：

```javascript
// 翻譯橋樑在背景幫你組裝出的設定檔，準備交給品管員（jQuery Validate）
rules: {
    Name: { required: true }
},
messages: {
    Name: { required: "請輸入姓名" }
}
```

這份「幕後組裝出來的設定檔」其實貫穿了整套機制 -- 後面章節講到自訂驗證規則時，你會看到同一套組裝邏輯再出現一次，到時候你就能秒懂。

### 步驟四：交接給品管員（執行）

翻譯橋樑把上一步組裝好的設定，直接呼叫並餵給 `jquery.validate.js`（品管員）。從現在開始，當使用者點擊「送出表單」或「離開文字方塊」時，就是品管員在負責攔截事件、檢查輸入值，並把「請輸入姓名」這段文字顯示在 `asp-validation-for`（或 `@Html.ValidationMessageFor`）對應的 `<span>` 裡面。

---
### ⚠️ 常見誤區：到底是誰在「掃描」data-val-* 屬性？

這裡要特別停下來提醒你，因為這是一個非常容易搞混、連寫筆記時都可能寫錯的地方。

你可能會在心裡這樣描述 `data-val="true"` 這個總開關：「**品管員**在掃描網頁時，只會去檢查帶有 `data-val="true"` 的輸入框」。

**這句話的角色設定錯了。** 正確的說法應該是：

> **翻譯橋樑（Unobtrusive）** 才是負責在網頁上到處掃描這張「總開關貼紙」的人，不是品管員。

完整的邏輯是這樣的：

1. **總開關貼紙（`data-val="true"`）**：代表「此欄位需要被關照」的識別證。
2. **翻譯橋樑出巡**：網頁一載入，翻譯橋樑會去找畫面上所有帶有 `data-val="true"` 的 `<input>`、`<select>` 等表單元素。
   - **沒有貼紙**：翻譯橋樑直接略過，連看都不看它身上其他的 `data-val-*` 屬性。品管員的規則本裡永遠不會出現這個欄位，於是這個欄位的驗證就被整個跳過了。
   - **有貼紙**：翻譯橋樑停下來，仔細看身上還有哪些附帶規則（例如 `data-val-required`），打包翻譯好之後，登記到**品管員的規則本**裡。
1. **品管員把關**：從這之後，使用者送出表單、品管員攔截 `submit` 事件、檢查輸入值、顯示錯誤訊息 -- 這整套「執行」的工作才是品管員的責任。

> 💡 **這個誤區為什麼很重要？** 因為它幫你建立一個更精確的心智模型：「掃描 + 翻譯 + 登記」是翻譯橋樑的工作，而且**只在網頁剛載入完成那一刻（或你手動呼叫 `parse()` 的那一刻）執行一次**；「拿著登記好的規則本去執行檢查」才是品管員的工作。這個分界線會在後面第九章（動態表單驗證重置）變得非常關鍵 -- 你會發現，動態新增的欄位之所以驗證不會生效，正是因為翻譯橋樑沒有機會再掃描它一次。

---
### 為什麼這套機制叫做「非侵入式」（Unobtrusive）？

順便解開一個常見的疑問：為什麼這支橋樑套件要取名叫 `unobtrusive`（不顯眼、非侵入式）？

對照一下，如果沒有這層翻譯橋樑，單純只用品管員（jQuery Validate）的話，前端 JS 必須自己寫死規則：

```javascript
// 侵入式的寫法：前端必須手動寫出邏輯，干預到 JavaScript 檔案裡
$("#myForm").validate({
    rules: {
        Name: { required: true }
    },
    messages: {
        Name: { required: "請輸入姓名" }
    }
});
```

這種寫法的缺點很明顯：後端 C# 改了驗證規則（例如把 `[Required]` 拿掉），前端這段 JS 也要跟著手動改，否則前後端規則就會不同步。

有了翻譯橋樑之後，你的 JavaScript 檔案裡**一行驗證規則都不用寫**。所有驗證邏輯統一由 C# Model 控管，MVC 負責產出 HTML，翻譯橋樑負責解讀並啟動品管員。因為它完全不需要你「侵入」JavaScript 檔案去手動設定規則，所以才被稱為「非侵入式（Unobtrusive）」。

---
## 四、data-val-* 屬性全面解析

### 輸入框（`<input>`）上的屬性

每個需要驗證的輸入框，都會被貼上這些屬性：

**`data-val="true"`（總開關）**：這是這個輸入框的「需要驗證」標記。翻譯橋樑在掃描網頁時，只會去處理帶有 `data-val="true"` 的輸入框，並把它登記進品管員的規則本。沒有這張貼紙，翻譯橋樑連看都不會看這個欄位身上其他的屬性，整個欄位的驗證就會被跳過。

**`data-val-[規則名稱]="錯誤訊息"`（具體規則與錯誤訊息）**：這是翻譯橋樑要讀取、翻譯、轉交給品管員的具體規則內容 -- 告訴品管員要查什麼，以及查到錯誤時要說什麼台詞。

### C# Annotation 與 HTML 屬性的完整對照表

● 翻譯後的 HTML 屬性（前端便利貼）都會加上 data-val="true" 總開關，後面的表格就不贅述

| **Data Annotation** （後端標準書寫）                                                                                                                                           | **產生的 HTML data-val-* 屬性** （前端便利貼）                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `[Required(ErrorMessage = "必填")]`                                                                                                                                      | `data-val-required="必填"`                                                                                                                        |
| `[EmailAddress(ErrorMessage = "Email格式錯誤")]`                                                                                                                           | `data-val-email="Email格式錯誤"`                                                                                                                    |
| `[StringLength(20, ErrorMessage = "不得超過20個字元")]`<br>_或_<br>`[StringLength(maximumLength: 20, ErrorMessage = "不得超過20個字元")]`                                             | `data-val-length="不得超過20個字元"`<br>`data-val-length-max="20"`                                                                                     |
| `[StringLength(10, MinimumLength = 5, ErrorMessage = "僅允許輸入5~20個字元")]`<br>_或_<br>`[StringLength(maximumLength: 20, MinimumLength = 5, ErrorMessage = "僅允許輸入5~20個字元")]` | `data-val-length="僅允許輸入5~20個字元"`<br>`data-val-length-max="20"`<br>`data-val-length-min="5"`                                                     |
| `[MinLength(5, ErrorMessage = "不得少於5個字元")]`                                                                                                                            | `data-val-minlength="不得少於5個字元"`<br>`data-val-minlength-min="5"`                                                                                 |
| `[MaxLength(20, ErrorMessage = "不得超過20個字元")]`                                                                                                                          | `data-val-maxlength="不得超過20個字元"`<br>`data-val-maxlength-max="20"`                                                                               |
| `[Range(1, 100, ErrorMessage = "超出範圍")]`                                                                                                                               | `data-val-range="超出範圍"`<br>`data-val-range-max="100"`<br>`data-val-range-min="1"`                                                               |
| `[RegularExpression(@"^\d+$", ErrorMessage = "限制輸入數字")]`                                                                                                               | `data-val-regex="限制輸入數字"`<br>`data-val-regex-pattern="^\d+$"`                                                                                   |
| `[Compare(nameof(Password), ErrorMessage = "密碼不一致")]`                                                                                                                  | `data-val-equalto="密碼不一致"`<br>`data-val-equalto-other="*.Password"`                                                                             |
| `[Remote(action: "IsExists", controller: "CheckAccount", AdditionalFields = "Id, Name", ErrorMessage = "此帳號已被使用")]`                                                    | `data-val-remote="此帳號已被使用"`<br>`data-val-remote-url="/CheckAccount/IsExists"` _(依路由設定自動生成)_<br>`data-val-remote-additionalfields="*.Id,*.Name"` |

> 💡 **你有沒有注意到**，有些規則會拆成兩個 HTML 屬性，一個存錯誤訊息，一個存數字參數？例如 `data-val-range="錯誤訊息"` 和 `data-val-range-min="1"`。這是因為 HTML 屬性只能存字串，數字參數需要另外一個屬性來承載。

---
### `data-val-equalto-other` 裡的 `*.` 是什麼意思？

你在看 `[Compare]` 翻譯後的 HTML 時，會看到 `data-val-equalto-other="*.Password"` 裡面有個 `*.`，這個星字號加小數點是「**相對路徑**」的意思。

它的意思是：「請在跟目前這個輸入框**相同層級**底下，尋找一個叫做 `Password` 的欄位」。就算你的 Model 包了好幾層物件，這套機制也能靠這個相對路徑精準找到正確的比對對象，不會找錯人。

---
### 顯示錯誤的 `<span>` 上的屬性

顯示錯誤訊息的 `<span>`（`asp-validation-for` 產生的）只需要兩個屬性：

**`data-valmsg-for="欄位名稱"`**：指定這個 `<span>` 是哪個輸入框的「專屬廣播喇叭」。當品管員發現錯誤，他會找到 `data-valmsg-for` 對應的喇叭，把錯誤訊息塞進去。

**`data-valmsg-replace="true"`**：控制錯誤訊息如何顯示。設為 `true` 時，每次貼上新的錯誤訊息前會先把舊的清掉，確保畫面只顯示最新的一條訊息，避免新舊訊息疊在一起造成版面混亂。ASP.NET Core 預設都會幫你產生 `true`。

---
### 實際產出的 HTML 長什麼樣？

當你在 View 裡面寫：

```html
<input asp-for="Email" class="form-control" />
<span asp-validation-for="Email" class="text-danger"></span>
```

因為 Model 上有 `[Required]` 和 `[EmailAddress]`，渲染後送到瀏覽器的 HTML 實際上會是這樣：

```html
<!-- 輸入框身上貼滿了檢查規則 -->
<input type="text" id="Email" name="Email" class="form-control"
       data-val="true"
       data-val-required="信箱不能為空"
       data-val-email="信箱格式不正確" />

<!-- 旁邊的 span 準備好接收錯誤訊息 -->
<span class="text-danger field-validation-valid"
      data-valmsg-for="Email"
      data-valmsg-replace="true"></span>
```

---
## 五、field-validation-error 和 input-validation-error 這些 CSS 是你的嗎？

當驗證失敗時，品管員除了把錯誤文字塞進 `<span>` 以外，還會做一件事：**替換 HTML 標籤上的 CSS class**，讓你可以透過樣式來讓畫面更直觀。

|元素|驗證通過時的 Class|驗證失敗時的 Class|
|---|---|---|
|`<span>`（廣播喇叭）|`field-validation-valid`|`field-validation-error`|
|`<input>`（輸入框）|`input-validation-valid`|`input-validation-error`|

這四個 class 名稱是寫死在 ASP.NET Core 的 Tag Helper 與 jQuery Unobtrusive Validation 底層程式碼裡的，當品管員偵測到驗證狀態改變時，就會自動在 `valid` 和 `error` 之間切換，**不需要你自己寫任何 JavaScript**。

---
**你可能會在 `wwwroot/css/site.css` 裡找不到這四個 class 的定義 -- 這是正常的。**

在早期的 .NET Framework 範本時代，微軟確實會把這四個 class 的樣式直接內建在 `Site.css` 裡。但較新的 ASP.NET Core 範本已將 UI 樣式大量交由 Bootstrap 負責，不再主動維護這四個 class 的 CSS 定義。

> 【補充說明】這點對你來說剛好是個對照組：GTS（.NET Framework 4.8 + Bootstrap 3.0.0）比較可能是早期範本年代留下的專案，site.css 裡或許還留著這四個 class 的定義；Portal（.NET Core 3.1 + Bootstrap 4.3.1）則更可能完全沒有，需要你自己補上。檢查的時候可以兩邊都搜一下 `field-validation-error` 字串，確認一下現況。

---
**你一定會問：我只要在 `<span>` 上加 `class="text-danger"`（Bootstrap），是不是也可以達到一樣的效果？**

可以，因為 `<span>` 沒有文字時看不見，有文字時 `text-danger` 讓它顯示為紅色，效果跟 `field-validation-error` 的樣式是相同的。

但如果你希望**輸入框本身在填錯時也顯示紅色框線**，那你就需要利用 `input-validation-error` 這個自動加上的 class 來寫樣式，而這個 class 是 Bootstrap 不會幫你處理的，**必須自己補在 `site.css` 裡**：

```css
.field-validation-error { color: red; }
.input-validation-error { border-color: red; }
```

> 💡 **結論**：`text-danger` 負責文字的顏色（Bootstrap 提供），`validation-error` 系列 Class 負責標記「目前處於錯誤狀態」（品管員自動加上）。兩者搭配使用，能讓錯誤提示既有紅字又有紅框，視覺效果最完整。

---
## 六、自訂驗證規則（Custom Attribute）：讓前後端都認識你自己發明的標籤

### 為什麼需要自訂？

內建的 Annotation 能搞定 80% 的日常需求，但遇到「台灣身分證字號驗證」、「密碼必須包含大寫字母和特殊符號」這種特殊業務規則，內建標籤就無能為力了。

如果我們直接把這些邏輯寫在 Controller 裡面，有兩個問題：Controller 會越來越肥，而且如果其他表單也需要同樣的驗證，程式碼就重複了。**自訂 Annotation 讓我們可以把驗證邏輯打包成一個隨插即用的標籤**。

### 第一步：打造後端驗證邏輯（繼承 `ValidationAttribute`）

所有驗證標籤（包含 `[Required]`、`[EmailAddress]` 等）的老祖宗都是 `ValidationAttribute`。繼承它就等於取得了成為合法驗證標籤的資格：

```csharp
// 引入 DataAnnotations 命名空間，這是所有驗證標籤（如 ValidationAttribute）的故鄉
using System.ComponentModel.DataAnnotations;
using System.Text.RegularExpressions;

// ValidationAttribute → 繼承「後端驗證標籤」的基礎類別，取得成為合法 Annotation 的資格
public class TaiwanIdAttribute : ValidationAttribute
{
	// ========================================== 
	// 第一部分：後端把關（負責伺服器端的驗證） 
	// ==========================================
    
    // 使用 override 覆寫基礎類別 ValidationAttribute 提供的 IsValid 方法 
    // 這是後端驗證的核心大腦 - 框架在完成 Model Binding 後會自動呼叫它 
    // value 參數：使用者填寫在輸入框裡的實際內容（原始證物） 
    // validationContext 參數：包含目前驗證環境的上下文資訊（此處暫不使用）
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        // 遇到空值，直接放行。
        // 重要原則：空值交給 [Required] 管，不是我們的責任。
        // 這樣設計讓 [TaiwanId] 可以用在「選填但如果填了就要符合格式」的情境。
        if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
        {
            return ValidationResult.Success;
        }

        string id = value.ToString();

        // 用正規表達式檢查身分證格式（首字大寫英文 + 數字1或2 + 8位數字）
        // @ 前綴 → 告訴 C# 不要解譯反斜線，讓 \d 直接交給 Regex 引擎處理
        // @ 放在字串前面，是告訴 C# 編譯器「這個字串請原封不動，不要解讀任何跳脫字元」。
        if (Regex.IsMatch(id, @"^[A-Z][1-2]\d{8}$"))
        {
            return ValidationResult.Success;
        }
        else
        {
            // ErrorMessage 就是你貼標籤時寫的 ErrorMessage 參數
            // ErrorMessage 是從基礎類別 ValidationAttribute 繼承來的公開屬性
            // 格式不符，建立一個新的「驗證失敗結果」物件並回傳
            return new ValidationResult(ErrorMessage ?? "身分證字號格式不正確");
        }
    }
}
```

貼到 Model 上的方式和內建 Annotation 完全一樣：

```csharp
public class RegisterViewModel
{
    [Required(ErrorMessage = "身分證字號必填")]
    [TaiwanId(ErrorMessage = "台灣身分證格式錯誤，請重新確認！")]
    public string IdNumber { get; set; }
}
```

---
### 第二步：讓前端也看得懂（實作 `IClientModelValidator`）

自訂標籤完成了**後端防線** -- 資料送到伺服器後，`ModelState.IsValid` 會正確攔截。

但前端的翻譯橋樑看到 `[TaiwanId]` 這張它不認識的標籤，會直接忽略它。使用者填錯身分證格式後，整個網頁會閃爍一下（送到後端再退回來），體驗很差。

要解決這個問題，我們需要讓 `TaiwanIdAttribute` 額外實作 `IClientModelValidator` 介面，手動告訴系統「在產生 HTML 時，請把這些便利貼貼到輸入框上」：

```csharp
// 引入 ModelBinding.Validation 命名空間，這是 IClientModelValidator 合約的所在地（ASP.NET Core 專屬）
using Microsoft.AspNetCore.Mvc.ModelBinding.Validation;
using System.ComponentModel.DataAnnotations;
using System.Collections.Generic;
using System.Text.RegularExpressions;

// 同時繼承 ValidationAttribute（後端把關）和實作 IClientModelValidator（前端便利貼）
// IClientModelValidator → 實作「前端產生 HTML 便利貼」的合約介面
public class TaiwanIdAttribute : ValidationAttribute, IClientModelValidator
{	
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        // ... 同上，後端邏輯不變 ...
        if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
            return ValidationResult.Success;

        return Regex.IsMatch(value.ToString(), @"^[A-Z][1-2]\d{8}$")
            ? ValidationResult.Success            
            : new ValidationResult(ErrorMessage ?? "身分證字號格式不正確");
    }

	// ========================================== 
	// 第二部分：前端翻譯（負責產生 HTML 的 data-val-* 屬性） 
	// ==========================================

    // 這個方法就是「發配便利貼的包裝站」
    // 實作 IClientModelValidator 介面規定的 AddValidation 方法 
    // 注意：這裡「不需要」寫 override，因為它是「實作介面」而不是「覆寫父類別方法」
    // context 參數：包含了準備貼在 HTML <input> 標籤上的所有屬性字典
    public void AddValidation(ClientModelValidationContext context)
    {
        // 貼上「安檢總電源開關」便利貼 
        // 前端翻譯橋樑只有看到 data-val="true" 才會掃描這個輸入框，並把後面的規則便利貼讀給品管員聽
        // 用 MergeAttribute 而非直接 Add，是為了防禦「已存在相同 Key」導致的例外
        MergeAttribute(context.Attributes, "data-val", "true");
        
        // 貼上專屬的 taiwanid 規則便利貼，並把錯誤訊息作為屬性值 
        // 前端 $.validator.unobtrusive.adapters.addBool("taiwanid") 會來讀取這個屬性
        MergeAttribute(context.Attributes, "data-val-taiwanid", ErrorMessage ?? "身分證格式錯誤");
    }

    // 安全地加入屬性，避免重複 Key 導致例外
    private bool MergeAttribute(IDictionary<string, string> attributes, string key, string value)
    {
	    // 防呆機制：先確認字典裡有沒有這個 key 
	    // 因為如果有其他 Annotation（如 [Required]）已經加入了 data-val="true"， 
	    // 再次呼叫 Add 會拋出 ArgumentException 讓程式當機
        if (attributes.ContainsKey(key))
            return false; // 已存在就不覆寫，直接跳過

        attributes.Add(key, value);
        return true;
    }
}
```

---
### 第三步：教導翻譯橋樑與品管員（JavaScript）

有了 `data-val-taiwanid` 這張便利貼，翻譯橋樑看到了，但它腦袋裡的預設字典裡沒有 `taiwanid` 這個單字，不知道該往哪裡轉譯；而品管員的規則手冊裡也沒有 `taiwanid` 這項檢查邏輯。所以這一步要分兩段，把兩邊都教會：

```javascript
// 第一步：教品管員「taiwanid 規則要怎麼檢查」（擴充執行邏輯）
$.validator.addMethod("taiwanid", function (value, element) {
    // this.optional(element)：「這個欄位是非必填、並且目前為空嗎？」
    // 如果是，直接放行，不執行後面的 Regex。
    // 注意：optional 這裡是「非必填」的意思，和 <select> 的 <option> 毫無關係。
    if (this.optional(element)) return true;
    return /^[A-Z][1-2]\d{8}$/.test(value);
});

// 第二步：教翻譯橋樑認得這個新單字，把 data-val-taiwanid 這個 HTML 屬性，
// 和上面定義的 taiwanid 規則「綁在一起」
// addBool 代表：只要 HTML 上出現 data-val-taiwanid 屬性（不管值是什麼），就觸發品管員的 "taiwanid" 規則
// 如果 Annotation 有帶參數的話（[TaiwanId(xxx, ErrorMessage="...")]），就要改用 addSingleVal、addMinMax
$.validator.unobtrusive.adapters.addBool("taiwanid");
```

> ⚠️ **大小寫陷阱**：C# 類別名稱是 `TaiwanId`（有大寫），但 HTML 屬性規範中 `data-*` 屬性會被強制轉成全部小寫（`data-val-taiwanid`）。在 JavaScript 裡綁定時，一定要全部用**小寫的 `"taiwanid"`**。如果寫成 `"TaiwanId"`，翻譯橋樑找不到對應的便利貼規則，驗證永遠不會觸發。

#### 為什麼這裡用 `addBool`，而不是別的 Adapter？

這點值得多解釋一句，因為它牽涉到你以後設計自己的驗證規則時該怎麼選擇。

`addBool` 適合的情境是「**開關型**」的規則——這個規則要嘛驗證、要嘛不驗證，不需要額外傳遞任何參數。身分證格式、台灣手機號碼格式都屬於這一類，因為正規表達式已經寫死在 JavaScript 裡了，HTML 屬性只需要負責「告訴翻譯橋樑要不要啟動這條規則」就夠了。

但如果你的自訂規則需要從 C# Annotation 帶參數過來（例如「字串長度區間」需要同時知道最大值跟最小值），單純的 `true/false` 就裝不下這些參數了，這時候就要改用 `addMinMax`（同時讀取 `-max` 跟 `-min` 兩個屬性）或 `addSingleVal`（讀取單一數值參數），才能把參數從 HTML 屬性裡正確抓出來餵給品管員。

#### 翻譯橋樑實際組裝出的設定檔長什麼樣子？

延續第三章學到的「幕後組裝」概念，我們來看看 `TaiwanId` 這個自訂規則，翻譯橋樑實際上組裝出來、轉交給品管員的設定檔長什麼樣子。

假設你的 `<input>` 是這樣：

```html
<input type="text" name="IdNumber"
       data-val="true"
       data-val-taiwanid="台灣身分證格式錯誤，請重新確認！" />
```

因為你已經用 `adapters.addBool("taiwanid")` 教過翻譯橋樑這個單字，它會在背景組裝出：

```javascript
// 翻譯橋樑準備交給品管員的設定檔
{
    rules: {
        "IdNumber": {
            taiwanid: true   // 因為是用 addBool，這裡的值固定是 true
        }
    },
    messages: {
        "IdNumber": {
            taiwanid: "台灣身分證格式錯誤，請重新確認！"  // 直接抓 HTML 屬性裡的文字
        }
    }
}
```

你可以看到，翻譯橋樑做的事情其實就是單純的「文字轉譯與歸納」：抓 `name` 當作規則對象、抓 `data-val-taiwanid` 知道要呼叫哪個規則（值固定為 `true`，因為是 `addBool`）、抓屬性裡的文字當作錯誤訊息。組裝好之後，整包丟給品管員，品管員從頭到尾都沒看過 HTML 上的 `data-val-*` 標籤，它只看到這份組裝好的乾淨設定檔。

### 整套自訂驗證的三個工作地點

完成了一個自訂驗證標籤，你實際上在三個不同的地方寫了程式碼：

在 **C# Attribute 類別（`.cs` 檔）** 裡，你繼承 `ValidationAttribute` 寫後端邏輯，並實作 `IClientModelValidator` 產生 HTML 便利貼。在 **Model（`.cs` 檔）** 裡，你把自訂標籤貼到屬性上，指定 `ErrorMessage`。在 **JavaScript 檔（`.js` 或 View 底部的 `<script>`）** 裡，你用 `addMethod` 定義前端檢查邏輯，並用 `adapters.addBool` 綁定 HTML 屬性和規則。

### 換個角度看：執行時間軸

上面是「在哪裡寫程式碼」的角度，現在換個角度，看看「實際跑起來的順序」是什麼，這樣你 debug 時會更有方向感：

1. **後端 C#**：你寫了一個自訂的 `ValidationAttribute`（`[TaiwanId]`）掛在 Model 的屬性上。
2. **View 產出 HTML**：MVC 框架把這個 Model 轉成 HTML，畫面上出現了帶有 `data-val-taiwanid="xxx"` 的 `<input>` 標籤。
3. **前端載入 JS**：你先用 `addMethod` 讓品管員學會了判斷身分證格式，接著用 `addBool` 把翻譯橋樑的字典更新。
4. **驗證啟動**：翻譯橋樑掃描整個網頁，看到 `data-val-taiwanid`，因為已經學過這個單字了，就順利把它翻譯成設定檔，轉交給品管員。使用者亂填身分證號碼時，就被成功擋下來了。

這就是為什麼寫自訂驗證時，`addMethod` 跟 `adapters.addBool` 這兩支 API 缺一不可的原因——少了任何一個，整條線都會斷掉，而且斷在不同地方（少了 `addMethod`，品管員根本不會驗證；少了 `addBool`，翻譯橋樑看不懂 HTML 屬性，永遠不會把規則交給品管員）。

---
## 七、純 jQuery 驗證 vs MVC Unobtrusive 驗證，差在哪？

你在開發過程中可能也遇過不用 `data-val-*` 屬性、直接在 JavaScript 裡設定規則的做法，這就是「純 jQuery 驗證」。

**純 jQuery 驗證**是直接呼叫 `.validate({ rules: {...} })`，把所有規則和錯誤訊息都寫在 JavaScript 裡：

```javascript
$("#myForm").validate({
    rules: {
        Email: { required: true, email: true },
        Password: { required: true, minlength: 6 }
    },
    messages: {
        Email: {
            required: "信箱不能是空的",
            email: "請輸入正確的信箱格式"
        }
    }
});
```

**MVC Unobtrusive 驗證**是微軟在純 jQuery 驗證之上加了一層翻譯橋樑，讓你只需要在 C# 寫一次規則，前後端都生效。你不需要手寫任何 JS 規則設定（這也正是第三章解釋過的「非侵入式」命名由來——對照一下就會發現，上面這段就是典型的「侵入式」寫法）。

### 我應該學哪一種？

在你的工作環境（C#、MVC、.NET Core / .NET Framework），**以 MVC Unobtrusive 驗證為主**，因為它讓你只維護一份規則，效率最高。

**純 jQuery 驗證則是你的備用武器**，在這個情境下使用它：當你需要根據使用者的操作「動態改變驗證規則」時（例如：使用者勾選了某個選項，另一個欄位才變成必填），C# 的 Annotation 幫不上忙，你就可以呼叫純 jQuery 的底層 API，例如 `$("#field").rules("add", { required: true })` 來動態加上規則。

> 💡 **最佳策略**：讓 C# 的 Annotation 處理 80% 的靜態格式驗證；遇到動態互動的 20%，用純 jQuery 的底層指令來補足。**但絕對不要在同一個欄位上同時用兩套系統設定規則**，它們會打架，驗證可能莫名其妙失效。

---
## 八、自訂 AJAX 送出時，前端防線會失效！

如果你把表單的送出按鈕從 `<button type="submit">` 改成普通按鈕，並在 jQuery 的 click 事件裡自己寫 `$.ajax()` 送出，那麼前端的驗證系統**不會自動觸發**，錯誤的資料會直接被送到後端。

原因是：品管員預設坐在表單的「正大門」（監聽 `submit` 事件）。如果你自己寫 AJAX 走了「側窗」，品管員根本不知道有東西被送出去。

解法是在發送 AJAX 之前，先手動呼叫 `$("form").valid()` 來叫醒品管員做一次全面檢查：

```javascript
$("#btnSave").click(function () {
    // 先手動叫品管員檢查，通過才繼續往下
    if (!$("form").valid()) return;

    $.ajax({
        // ... 發送資料到後端 ...
    });
});
```

---
## 九、動態表單的驗證重置：removeData 與 parse 組合拳

### 問題：動態新增的欄位為什麼沒有驗證？

**翻譯橋樑只會在網頁剛載入完成的那一瞬間，掃描畫面上所有的 `data-val-*` 便利貼，並把它們登記到品管員的規則本裡**。如果你後來透過 AJAX 或 jQuery 把新的 `<input>` 插入畫面，翻譯橋樑根本沒注意到它的存在、沒幫它登記，品管員的規則本裡自然沒有這筆記錄，他當然不會去檢查它。

### 解法：砍掉重練

當你動態更新了表單的 HTML 結構後，需要執行「重置驗證三步驟」讓翻譯橋樑重新掃描、品管員更新他的規則本：

```javascript
var $form = $("#createForm");

// 步驟一：清除表單的驗證快取（順序絕對不能顛倒！） 
// 移除 jQuery Validation 與 Unobtrusive Validation 的初始化記憶 
// 若不清除，兩者會認為已初始化完畢而略過重新解析
$form.removeData('validator').removeData('unobtrusiveValidation');

// 步驟二：重新解析表單驗證規則 
// 掃描表單內所有 data-val-* 屬性，重新建立 unobtrusive validation 驗證規則
$.validator.unobtrusive.parse($form);

// 步驟三：清除畫面上的錯誤提示（選擇性） 
// 重置所有驗證狀態，讓表單回到初始狀態
$form.validate().resetForm();
```

為什麼有兩個 `removeData`？因為有兩個角色各自有一本記憶：`validator` 是核心品管員（jQuery Validation）的記憶，`unobtrusiveValidation` 是微軟翻譯橋樑（Unobtrusive）的記憶。如果只清掉其中一個，另一個還記得「已初始化過了」，就會拒絕重新建立名冊。

> ⚠️ **黃金順序**：先 `removeData`（銷毀），再 `parse`（重建），最後 `resetForm`（清理版面）。絕對不能顛倒 -- 如果你先 `parse` 再 `removeData`，最後手上一本名冊都沒有，驗證系統全面癱瘓。

### 什麼時候不需要重置？

移除 HTML 欄位和隱藏 HTML 欄位是完全不同的兩件事，你需要分辨清楚：

**用 `.remove()` 把欄位刪除**：品管員名冊上找不到那個 `<input>` 了，自然不會去驗證它，通常不需要執行重置組合拳。

**用 `.hide()` 把欄位隱藏（`display: none`）**：`<input>` 標籤其實還在畫面上，品管員的規則本裡還記著它！如果那是必填欄位，使用者會遇到「表單送不出去，卻看不到哪裡有紅字（因為被隱藏了）」的糟糕體驗。建議改用 `.remove()` 刪除欄位，或在送出前動態用 `rules("remove")` 拔掉規則。

### 完整範例：AJAX 載入新內容後的標準流程

```javascript
$("#someButton").click(function () {
    // 1. 動態改變表單內容（例如透過 AJAX 載入 Partial View）
    $("#createForm").html(newContent);

    // 2. 如果有自訂驗證方法，在 parse 之前先定義
    $.validator.addMethod("taiwanmobile", function (value, element) {
        return this.optional(element) || /^09\d{8}$/.test(value);
    });
    $.validator.unobtrusive.adapters.addBool("taiwanmobile");

    // 3. 重置並重新初始化驗證（黃金順序）
    var $form = $("#createForm");
    $form.removeData('validator').removeData('unobtrusiveValidation');
    $.validator.unobtrusive.parse($form);
    $form.validate().resetForm();
});

// 4. 在送出時檢查驗證狀態（注意！不要在重置後立刻呼叫 .valid()，
//    否則畫面會立刻爆出一堆必填紅字，使用者體驗很差）
$("#btnSubmit").click(function () {
    if (!$("#createForm").valid()) return;

    $.ajax({ /* ... */ });
});
```

---
## 十、在 MVC 環境動態移除或加回驗證規則的正統做法

### 用純 jQuery 的 `rules("remove")` 可以嗎？

可以用，但有隱患。當你呼叫 `$("#Email").rules("remove", "required")` 時，你只是把品管員腦袋裡的規則刪掉了，但 HTML 標籤上的 `data-val-required` 便利貼**還在**。下次如果有人呼叫了 `parse()`，翻譯橋樑會重新掃描這張便利貼，規則就像殭屍一樣復活了。

### 正統的 MVC 做法

在 MVC 環境裡，**HTML 屬性才是真正的主人**。要動態移除規則，應該先把 HTML 便利貼撕掉，然後重新 `parse`：

```javascript
// 動態拔除 Email 的必填驗證
$("#Email").removeAttr("data-val-required");
// 如果整個欄位完全不需要驗證，連總開關都撕掉
// $("#Email").removeAttr("data-val");

// 重置讓翻譯橋樑重新掃描、更新品管員的規則本
var $form = $("#createForm");
$form.removeData("validator").removeData("unobtrusiveValidation");
$.validator.unobtrusive.parse($form);
```

```javascript
// 動態加回 Email 的必填驗證
$("#Email").attr("data-val", "true");
$("#Email").attr("data-val-required", "這個欄位現在變成必填囉！");

// 同樣要重置讓翻譯橋樑重新掃描、更新品管員的規則本
var $form = $("#createForm");
$form.removeData("validator").removeData("unobtrusiveValidation");
$.validator.unobtrusive.parse($form);
```

---
## 十一、純 jQuery 驗證的重置方式（作為對比參考）

如果你在某些非 MVC 環境使用純 jQuery 驗證，重置的方式截然不同，這裡作為對比說明。

純 jQuery 驗證沒有「翻譯橋樑」，所有規則都直接存在品管員自己的 JS 物件裡，重置只需要操作這一個物件：

```javascript
// 局部調整：只對某個欄位加上新規則
$("#newPhone").rules("add", {
    required: true,
    digits: true,
    messages: { required: "手機號碼必填", digits: "只能輸入數字" }
});

// 移除某個規則
$("#Email").rules("remove", "required");

// 大範圍更新：砍掉整個驗證器，重新設定
// 注意！純 jQuery 的 .validate() 是初始化指令，重複呼叫會被忽略，
// 一定要先 .destroy() 才能重新初始化
$("#myForm").validate().destroy();

$("#myForm").validate({
    rules: {
        newEmail: { required: true },
        newPhone: { required: true, digits: true }
    }
});
```

|操作情境|MVC Unobtrusive（翻譯橋樑）|純 jQuery 驗證|
|---|---|---|
|全面重置|`removeData('validator').removeData('unobtrusiveValidation')` 後 `parse()`|`.validate().destroy()` 後重新 `.validate({ rules: {...} })`|
|清除紅字|`.validate().resetForm()`|`.validate().resetForm()`（兩者相同）|
|動態新增欄位規則|操作 HTML `attr`，再重新 `parse`|`.rules("add", {...})`|
|動態移除欄位規則|操作 HTML `removeAttr`，再重新 `parse`|`.rules("remove", "ruleName")`|

---
## 十二、完整 HTML 範例：整合所有學過的驗證技巧

以下是一個把本文所有概念整合在一起的完整表單範例，包含各種內建 `data-val-*` 屬性、自訂驗證方法，以及正確的腳本載入順序：

```html
<style>
    .field-validation-error { color: red; }
    .input-validation-error { border: 1px solid red; }
</style>

<!-- 必要的腳本：jQuery → jQuery Validate → Unobtrusive -->
<script src="jquery.min.js"></script>
<script src="jquery.validate.min.js"></script>
<script src="jquery.validate.unobtrusive.min.js"></script>

<form id="registrationForm" novalidate>
    <!-- 必填 + 長度驗證 -->
    <div class="mb-3">
        <label for="username">用戶名稱</label>
        <input type="text" id="username" name="username"
               data-val="true"
               data-val-required="請輸入用戶名稱"
               data-val-length="用戶名稱長度需要在 2-20 個字符之間"
               data-val-length-min="2"
               data-val-length-max="20" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="username"
              data-valmsg-replace="true"></span>
    </div>

    <!-- Email 格式驗證 -->
    <div class="mb-3">
        <label for="email">電子郵件</label>
        <input type="email" id="email" name="email"
               data-val="true"
               data-val-required="請輸入電子郵件"
               data-val-email="請輸入有效的電子郵件地址" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="email"
              data-valmsg-replace="true"></span>
    </div>

    <!-- 正規表達式驗證 -->
    <div class="mb-3">
        <label for="phone">電話號碼</label>
        <input type="tel" id="phone" name="phone"
               data-val="true"
               data-val-required="請輸入電話號碼"
               data-val-regex="請輸入 10 位數字的電話號碼"
               data-val-regex-pattern="^[0-9]{10}$" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="phone"
              data-valmsg-replace="true"></span>
    </div>

    <!-- 自訂驗證：台灣手機號碼 -->
    <div class="mb-3">
        <label for="mobile">手機號碼</label>
        <input type="tel" id="mobile" name="mobile"
               data-val="true"
               data-val-required="手機號碼為必填欄位"
               data-val-taiwanmobile="請輸入有效的台灣手機號碼（09 開頭，共 10 碼）" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="mobile"
              data-valmsg-replace="true"></span>
    </div>

    <!-- 數值範圍驗證 -->
    <div class="mb-3">
        <label for="age">年齡</label>
        <input type="number" id="age" name="age"
               data-val="true"
               data-val-required="請輸入年齡"
               data-val-range="年齡必須在 18-120 歲之間"
               data-val-range-min="18"
               data-val-range-max="120" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="age"
              data-valmsg-replace="true"></span>
    </div>

    <!-- 確認密碼（equalto）-->
    <div class="mb-3">
        <label for="password">密碼</label>
        <input type="password" id="password" name="password"
               data-val="true"
               data-val-required="請輸入密碼"
               data-val-length="密碼長度至少 8 個字符"
               data-val-length-min="8" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="password"
              data-valmsg-replace="true"></span>
    </div>

    <div class="mb-3">
        <label for="confirmPassword">確認密碼</label>
        <input type="password" id="confirmPassword" name="confirmPassword"
               data-val="true"
               data-val-required="請確認密碼"
               data-val-equalto="兩次輸入的密碼不一致"
               data-val-equalto-other="*.password" />
        <span class="text-danger field-validation-valid"
              data-valmsg-for="confirmPassword"
              data-valmsg-replace="true"></span>
    </div>

    <button type="submit">註冊</button>
</form>

<script>
$(document).ready(function () {
    var $form = $("#registrationForm");

    // === 定義自訂驗證方法（必須在 parse 之前定義！）===
    $.validator.addMethod("taiwanmobile", function (value, element) {
        // this.optional(element) 讓選填欄位為空時直接通過，不跑後面的正規表達式
        return this.optional(element) || /^09\d{8}$/.test(value);
    }, "請輸入有效的台灣手機號碼");

    // 把自訂方法與 data-val-taiwanmobile 屬性綁定在一起
    $.validator.unobtrusive.adapters.addBool("taiwanmobile");
    // =====================================================

    // 重置並重新初始化驗證（確保自訂方法有正確被套用）
    $form.removeData('validator').removeData('unobtrusiveValidation');
    $.validator.unobtrusive.parse($form);
    $form.validate().resetForm();

    // 如果是用普通按鈕送出（非 type="submit"），要手動觸發驗證
    // $form.on("submit", function (e) {
    //     return $(this).valid();
    // });
});
</script>
```
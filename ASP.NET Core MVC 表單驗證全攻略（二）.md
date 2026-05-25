---
date: 2026-05-25 07:53
title:
aliases:
tags:
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[ASP.NET Core MVC 表單驗證全攻略（一）]]
#### 📑 [[ASP.NET Core MVC 表單驗證全攻略（三）]]

---
## 一、ModelState 是什麼？為什麼需要它？

在前一篇，我們知道 Controller 裡有個 `ModelState.IsValid` 可以一眼判斷表單有沒有問題。但 `ModelState` 本身是什麼？它裡面又裝了什麼？大多數人到這個問題就卡住了。

讓我們先回到問題本身：當使用者按下「送出」，瀏覽器把一堆純文字（字串）傳到伺服器，系統要做兩件事——

**第一件事，型別轉換（Model Binding）**：把文字轉成 C# 物件。例如使用者在生日欄位輸入了 `"2000-01-01"`，系統要把它從字串變成 `DateTime` 型別；在年齡欄位輸入了 `"25"`，要變成 `int`。

**第二件事，規則驗證（Validation）**：轉換成功後，再去比對 Model 上面貼的 `[Required]`、`[EmailAddress]` 等 Annotation 規則。

這兩個過程中，可能發生各種狀況。`ModelState` 就是用來**統一收集、記錄這整個過程中所有成功與失敗的細節**，讓你的 Controller 能夠根據這份「總結報告」決定下一步怎麼做。

---
## 二、生活比喻：海關的安檢紀錄本 🛂

把 `ModelState` 想像成安檢員手上拿著的「安檢紀錄本」，整個流程有兩道關卡：

**第一關（型別轉換）**：檢查行李能不能放進 X 光機。如果旅客帶了一頭大象（字串 `"abc"` 想塞進 `int` 欄位），塞不進去，安檢員就在紀錄本記下一筆錯誤。

**第二關（規則驗證）**：行李順利進去了，再檢查有沒有違禁品（比對 Annotation 規則）。發現有剪刀，再記下一筆錯誤。

海關主管（Controller）只需要看一眼紀錄本的封面（`ModelState.IsValid`），就知道這個人能不能放行。如果需要知道細節，才打開紀錄本一頁一頁翻。

---
## 三、ModelState 的內部結構：從類別圖看清楚

`ModelState` 的底層是一個字典（`ModelStateDictionary`），以「欄位名稱」作為 Key，每個 Key 對應一份「詳細體檢報告」。我們用簡化版的 C# 類別來把它的樹狀結構攤開來看：

```
ModelStateDictionary（整個檔案櫃 / 海關總紀錄本）
│
├── IsValid（bool）→ 主管看一眼：檔案櫃裡是不是完全沒有違規紀錄？
├── ErrorCount（int）→ 所有錯誤的總件數
├── Root（ModelStateEntry）→ 總申報單，代表整份表單本身（Key 為 ""）
├── Keys（清單）→ 所有抽屜外面的「標籤貼紙」（例如："Age"、"Email"）
└── Values（清單）→ 所有抽屜裡面的「詳細體檢報告」
    │
    ├── ModelStateEntry（標籤為 "Email" 的專屬抽屜）
    │   ├── AttemptedValue → "abc"（案發現場：使用者亂填的字串）
    │   ├── RawValue → 原始 HTTP 傳入的結構（通常是字串陣列）
    │   ├── ValidationState → Invalid（抽屜的狀態紅章）
    │   └── Errors（ModelErrorCollection / 裝滿錯誤的資料夾）
    │       ├── ModelError → { ErrorMessage: "信箱格式不正確" }
    │       └── ModelError → { ErrorMessage: "此信箱已被註冊" }
    │
    └── ModelStateEntry（標籤為 "Age" 的專屬抽屜）
        ├── AttemptedValue → "十歲"
        ├── ValidationState → Invalid
        └── Errors
            └── ModelError → { Exception: 發生轉型失敗的系統例外 }
```

三層結構的分工很清楚：`ModelStateDictionary` 管理整份表單，`ModelStateEntry` 管理單一欄位，`ModelError` 管理單一欄位裡的每一條具體錯誤。

---
## 四、逐一解析每個重要屬性

### 4.1 `ModelState.IsValid`——主管看一眼的結論

```csharp
if (!ModelState.IsValid)
{
    return View(model);
}
```

`IsValid` 是布林值，只要整本紀錄本裡（包含 Root 和所有子欄位）沒有任何一筆錯誤，它就是 `true`。這是 Controller 最常使用的屬性，通常放在 Action 的第一行。

---
### 4.2 `ModelState.ErrorCount`——全域錯誤計數器

`ErrorCount` 回傳一個整數，代表目前整張表單中所有錯誤的總件數。它在底層會走訪 Root 和每一個子欄位的 Errors，把所有 `ModelError` 的數量加總起來。

這個屬性在幾個場景特別有用：

**API 回應設計**：在開發 Web API 時，讓前端知道有幾條錯誤需要處理：

```csharp
if (!ModelState.IsValid)
{
    return BadRequest(new {
        Message = $"表單驗證失敗，總共發現了 {ModelState.ErrorCount} 個錯誤。",
        Details = ModelState
    });
}
```

**系統日誌**：記錄錯誤數量，方便維運人員判斷這是使用者手滑（`ErrorCount = 1`），還是惡意攻擊（`ErrorCount = 20`）。

**單元測試**：精準驗證驗證邏輯是否如預期：

```csharp
Assert.False(modelState.IsValid);
Assert.Equal(1, modelState.ErrorCount); // 預期剛好只有 1 個錯誤
```

---
### 4.3 `ModelState.Root`——整份表單的「總申報單」

`Root` 是 `ModelStateDictionary` 裡一個特殊的 `ModelStateEntry`，它的 Key 是空字串 `""`，代表的是「整張表單本身」，而不是任何特定欄位。

你每次呼叫 `ModelState.AddModelError(string.Empty, "某段訊息")` 時，這個訊息其實就是被塞進 `Root.Errors` 裡面的。而前端的 `asp-validation-summary="ModelOnly"` 大螢幕，就是去讀取 `Root.Errors` 的內容來顯示。

這樣你就能理解為什麼 `ModelOnly` 只顯示整體性錯誤了——因為它只看 Root 這一層，不看底下的子欄位。

---
### 4.4 `ModelStateEntry.ValidationState`——抽屜的狀態紅章

很多人第一次看到 `ValidationState` 會直覺地問：「這應該是 `bool` 吧？」

答案是不行，因為驗證結果不是只有「通過」和「失敗」兩種狀態。現實中還有「還沒輪到它」和「這個欄位不需要驗證」的情況，一個 `bool` 根本表達不完，所以微軟設計成列舉（Enum）：

```csharp
public enum ModelValidationState
{
    Unvalidated = 0, // 資料剛綁定好，尚未開始驗證（排隊中）
    Invalid     = 1, // 檢查完畢，發現至少一個錯誤
    Valid       = 2, // 檢查完畢，完全沒有錯誤
    Skipped     = 3  // 系統判定這個欄位不需要被驗證，直接跳過
}
```

`Skipped` 這個狀態通常出現在你在 Model 屬性上貼了 `[ValidateNever]` 標籤的時候——它告訴系統「請直接跳過這個欄位，不要驗證它」，就像享有外交豁免權的大使，行李不用接受安檢。

> ⚠️ **踩坑警告**：不要直接寫 `if (ModelState["Email"].ValidationState == true)`——這樣寫絕對編譯錯誤。正確寫法是用 Enum 比對：
> 
> ```csharp
> if (ModelState["Email"].ValidationState == ModelValidationState.Valid)
> ```
> 
> 但即使是這個寫法，也非常危險。如果前端根本沒有送出 `Email` 欄位，`ModelState["Email"]` 會回傳 `null`，接著呼叫 `.ValidationState` 就會爆出 `NullReferenceException`，整個頁面崩潰。在實務上，我們幾乎只使用最外層的 `ModelState.IsValid`，而不會去底層針對單一欄位判斷狀態。

---
### 4.5 `AttemptedValue` vs `RawValue`——保留案發現場的兩個屬性

這兩個屬性看起來很像，但有明確的差異：

**`AttemptedValue`（嘗試值）**：型別是 `string`，永遠是純字串。這是 HTML 畫面產生器（Tag Helper）用來把「使用者填錯的字」重新貼回輸入框的依據。例如使用者在年齡欄位填了 `"十歲"`，轉型為 `int` 失敗，`AttemptedValue` 就保留了 `"十歲"` 這個字串，讓畫面退回後輸入框裡不會變空白。

**`RawValue`（原始值）**：型別是 `object`，在 .NET Core 通常是字串陣列（`string[]` 或 `StringValues`），保留了原始 HTTP 請求的資料結構。例如使用者勾選了多個 Checkbox，`RawValue` 會保留完整的陣列 `["看書", "打球"]`，而 `AttemptedValue` 則是逗號串接後的字串 `"看書,打球"`。

|屬性|型別|誰會用到它|實務用途|
|---|---|---|---|
|`AttemptedValue`|`string`|HTML Tag Helper|把使用者填錯的字串還原到輸入框|
|`RawValue`|`object`（通常是字串陣列）|C# 底層轉換引擎|寫自訂 Model Binder 時使用|

> 💡 **實務準則**：在日常的 CRUD 開發中，這兩個屬性你幾乎不需要主動去碰。它們是 MVC 框架幕後自動使用的工具。你真的要動到它們的情境，只有寫自訂 Model Binder，或是記錄惡意攻擊的 Log 時。

---
### 4.6 `ModelErrorCollection`——為什麼是集合，而不是單一字串？

`ModelStateEntry.Errors` 是一個集合（`ModelErrorCollection`，繼承自 `Collection<ModelError>`），而不是單一字串。

這樣設計是因為一個欄位可能同時違反多條規則。例如密碼欄位同時貼了 `[Required]` 和 `[MinLength(8)]` 和 `[RegularExpression(...)]`，如果使用者什麼都沒填，理論上三條規則都違反了（雖然實務上通常 `[Required]` 先擋下來就短路了）。

`ModelErrorCollection` 雖然名字特別，但本質就是一個可以 `foreach` 的集合。微軟把它包裝成專屬類別而不是直接用 `List`，是為了框架內部管理的安全性——可以在裡面加上特製方法，並避免外部程式碼隨意修改。

---
## 五、安全地存取字典：`TryGetValue` vs `ContainsKey`

`ModelState` 底層是一個字典，字典有個著名的地雷：如果你直接用 `[]` 索引子去取一個不存在的 Key，程式不會回傳 `null`，而是直接拋出 `KeyNotFoundException`，整個請求崩潰。

因此，存取字典要養成「先確認、再取值」的習慣。C# 提供了兩個工具：

**`ContainsKey`（先問有沒有）**：只確認 Key 是否存在，不取值。適合你只需要確認欄位存不存在、完全不需要看裡面內容的情境。

```csharp
if (ModelState.ContainsKey("Email"))
{
    // 只有確認存在才進入這裡，但還沒取值
}
```

**`TryGetValue`（試著拿拿看）**：一次完成「確認存在 + 取值」兩件事，效能更好，因為字典只被搜尋了一次（`ContainsKey` 再 `[]` 是搜尋兩次，效能較差）。

```csharp
// 現代 C# 7.0+ 的簡潔寫法（inline out variable）
if (ModelState.TryGetValue("Email", out var entry))
{
    // entry 就是 ModelStateEntry，可以直接使用
    string firstError = entry.Errors.FirstOrDefault()?.ErrorMessage;
}
```

> 💡 **決策口訣**：「要拿值就用 `TryGetValue`；只問在不在就用 `ContainsKey`。」
> 
> ⚠️ **效能地雷**：很多人直覺地寫 `if (dict.ContainsKey("Email")) { var x = dict["Email"]; }`——這樣字典被搜尋了兩次，資料量大時是隱形的效能殺手，請改用 `TryGetValue`。

---
## 六、操作 ModelState 的常用方法

### `ModelState.AddModelError(key, message)`

把一筆錯誤手動寫進紀錄本。`key` 填欄位名稱就綁定到特定欄位，填空字串就是整體性錯誤（存到 Root）：

```csharp
// 特定欄位錯誤
ModelState.AddModelError("Account", "這個帳號已被使用");
ModelState.AddModelError("Email", "此信箱已被其他帳號使用");

// 整體性錯誤（顯示在 ModelOnly 大螢幕）
ModelState.AddModelError(string.Empty, "伺服器處理失敗，請稍後再試");
```

---
### `ModelState.Remove(key)` vs `ModelState.Clear()`

有時候業務需求需要略過某個欄位的驗證，這時你需要把那個欄位的記錄從 `ModelState` 裡清除。

`ModelState.Remove("Age")`：把指定 Key 的整個抽屜從字典裡移除。之後 `ModelState.IsValid` 不會再把它計入。

`ModelState.Clear()`：清除所有記錄，包含所有欄位的錯誤和驗證狀態，整本紀錄本歸零。

> ⚠️ **不要直接呼叫 `ModelState["Age"].Errors.Clear()`**：這個做法很危險！`Errors.Clear()` 只清除了錯誤文字，但該欄位的 `ValidationState` 已經被蓋上了 `Invalid` 的紅章，`ModelState.IsValid` 依然可能判定為失敗。請永遠使用 `ModelState.Remove()` 或 `ModelState.Clear()` 這兩個框架層級的方法，它們是「砍掉整個抽屜」而不是「只擦掉抽屜裡的文字」。

---
### `ModelState.ClearValidationState(key)`

把指定欄位的錯誤清空，但把狀態重置回 `Unvalidated`（排隊中），而不是直接刪除這個抽屜。適用於你打算後續手動重新執行驗證的進階情境。

---
## 七、嵌套物件的 Key 命名規則

如果你的 ViewModel 裡包著另一個物件，ModelState 的 Key 會用「點（`.`）」連接，和 C# 呼叫物件屬性的寫法一樣：

```csharp
public class RegisterViewModel
{
    public Address UserAddress { get; set; }
}

public class Address
{
    [Required(ErrorMessage = "城市必填")]
    public string City { get; set; }
}
```

當 `City` 驗證失敗時，在 `ModelState` 字典裡記錄這筆錯誤的 Key 是 `"UserAddress.City"`。前端畫面的 `asp-validation-for="UserAddress.City"` 就是靠這個命名規則來精準對應的。

---
## 八、實戰演練：從一次完整的表單提交看 ModelState 如何填入

讓我們模擬一個情境：年齡欄位預期是 `int`，使用者填了 `"十歲"`；信箱欄位使用者填了 `"abc"`（格式不對）。

當這份表單送到 Controller，`ModelState` 內部的狀態會是這樣：

| Key       | AttemptedValue | ValidationState | Errors 的 ErrorMessage                  | 發生階段                  |
| --------- | -------------- | --------------- | -------------------------------------- | --------------------- |
| `"Age"`   | `"十歲"`         | `Invalid`       | "The value '十歲' is not valid for Age." | 第一關：型別轉換失敗（字串無法轉 int） |
| `"Email"` | `"abc"`        | `Invalid`       | "信箱格式不正確"                              | 第二關：Annotation 規則失敗   |

系統先嘗試把 `"十歲"` 塞進 `int Age`，失敗，建立 Key 為 `"Age"` 的抽屜，把原始字串存進 `AttemptedValue`，並在 `Errors` 寫下型別錯誤。接著處理 `Email`，字串轉換成功，但進入第二關後發現 `[EmailAddress]` 規則不通過，建立 Key 為 `"Email"` 的抽屜並記下格式錯誤。最外層的 `ModelState.IsValid` 看到有抽屜有錯，自動變成 `false`。

---
## 九、`ModelState.Clear()` 解決「畫面改不掉」的經典地雷

學到這裡，你已經理解了一個非常重要的幕後機制：HTML 的 `<input asp-for="Age" />` 在產生畫面時，會先看 `ModelState["Age"].AttemptedValue` 裡有沒有值，如果有，**優先顯示 AttemptedValue 的字串**，完全忽略你在 Model 裡設定的值。

這正是下面這個讓新手抓狂的 Bug 的根源：

```csharp
[HttpPost]
public IActionResult Edit(User model)
{
    if (!ModelState.IsValid)
    {
        // 💣 你以為這樣會讓畫面顯示 18，但它還是顯示 "ABC"！
        model.Age = 18;
        return View(model);
    }
    // ...
}
```

因為 `ModelState["Age"].AttemptedValue` 裡還存著 `"ABC"`，`Tag Helper` 就把 `"ABC"` 印到輸入框上，完全無視你改的 `model.Age = 18`。

解法是在改值之前，先用 `ModelState.Clear()` 清空整本紀錄本：

```csharp
if (!ModelState.IsValid)
{
    ModelState.Clear(); // 把所有 AttemptedValue 清空
    model.Age = 18;     // 現在 Tag Helper 找不到 AttemptedValue，才會去讀 model.Age
    return View(model);
}
```

清空後，`Tag Helper` 找不到任何 `AttemptedValue`，就會乖乖去讀你設定的 `model.Age`，畫面就會正確顯示 `18` 了。

---
## 十、完整 ModelState 類別結構參考

以下是 `ModelStateDictionary` 相關類別的簡化版結構，適合作為日後查閱的速查參考：

```csharp
// 整個表單的驗證總覽（海關總紀錄本）
public class ModelStateDictionary : IEnumerable<KeyValuePair<string, ModelStateEntry>>
{
    public bool IsValid { get; }              // 整份表單是否全部通過？
    public int ErrorCount { get; }            // 所有錯誤的總件數
    public ModelStateEntry Root { get; }      // 代表整份表單本身（Key 為 ""）
    public IEnumerable<string> Keys { get; }  // 所有欄位名稱
    public IEnumerable<ModelStateEntry> Values { get; } // 所有欄位的體檢報告

    // 手動新增錯誤（key 填欄位名稱，或空字串代表整體錯誤）
    public void AddModelError(string key, string errorMessage);

    // 完全清空所有記錄
    public void Clear();

    // 確認 Key 是否存在（只問在不在，不取值）
    public bool ContainsKey(string key);

    // 移除指定欄位的整個抽屜
    public bool Remove(string key);

    // 安全取值（推薦用這個，避免 NullReferenceException）
    public bool TryGetValue(string key, out ModelStateEntry value);

    // 索引器（有地雷！Key 不存在時回傳 null，不是拋例外）
    public ModelStateEntry this[string key] { get; }
}

// 單一欄位的詳細體檢報告
public class ModelStateEntry
{
    public string AttemptedValue { get; }         // 使用者送出的原始字串
    public object RawValue { get; }               // HTTP 傳來的原始結構（通常是字串陣列）
    public ModelValidationState ValidationState { get; } // 該欄位的驗證狀態
    public ModelErrorCollection Errors { get; }   // 這個欄位犯了哪些錯
}

// 單一條錯誤的內容
public class ModelError
{
    public string ErrorMessage { get; }  // 人類可讀的錯誤訊息文字
    public Exception Exception { get; }  // 若是型別轉換失敗，例外物件在這裡
}

// 驗證狀態的列舉定義
public enum ModelValidationState
{
    Unvalidated = 0, // 尚未開始驗證
    Invalid     = 1, // 驗證失敗
    Valid       = 2, // 驗證通過
    Skipped     = 3  // 跳過驗證（例如加了 [ValidateNever]）
}
```
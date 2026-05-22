---
date: 2026-05-22 11:29
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
#### 📑 [[]]

---
## 前言：網頁是怎麼被「組裝」出來的？

還記得第一次打開 MVC 專案，看到一堆 `.cshtml` 檔案時的困惑嗎？`_Layout.cshtml`、`Index.cshtml`、`_ViewStart.cshtml`⋯⋯這些檔案各自扮演什麼角色，最後又是怎麼拼在一起變成一個完整網頁的？

---
## 一、先看懂問題，才能欣賞解法

### 1.1 JavaScript 放哪裡，差很多

在聊 `@section Scripts` 之前，我們得先理解一個底層的網頁效能原則：**JavaScript 要放在頁面最底部**。

為什麼？因為瀏覽器讀取 HTML 是**由上往下、一行一行執行**的。如果你把 `<script>` 標籤放在 `<body>` 的最上面，瀏覽器遇到它就會先去下載並執行這段程式碼，**整個畫面的渲染會在這段時間完全暫停**。使用者開啟頁面，看到的就是一片空白，體驗非常差。

所以，業界的標準做法是：把 JavaScript 的 `<script>` 標籤放在 `</body>` 結束標籤的正上方。

### 1.2 但在 MVC 架構下，這件事有點麻煩

MVC 的頁面是被拆開管理的。一個完整的頁面，通常由兩個部分組成：

- **主版面（`_Layout.cshtml`）**：放置全站共用的 HTML 骨架，包含導覽列、頁尾，當然還有放在底部的 jQuery 等核心 JS 檔案。
- **子頁面（`Index.cshtml` 等）**：放置各個頁面專屬的內容。

問題來了：**我的 JavaScript 邏輯跟著子頁面走，但這段程式碼必須被放到主版面的最底部才對。**

子頁面沒辦法直接跳進主版面的 HTML 去塞東西，這兩個檔案是分離的。

這就是 `@section Scripts` 存在的原因。

---
## 二、`@section Scripts`：跨頁面的快遞服務

### 2.1 核心概念：收件區 × 寄件包裹

`@section Scripts` 的運作機制，我喜歡用「快遞服務」來比喻：

> 主版面（`_Layout.cshtml`）就像是演唱會舞台，舞台最後方畫了一個「道具存放區」。每一個子頁面就像不同的表演者。表演者可以在自己的頁面裡，把需要的專屬道具（JS 程式碼）打包在一個叫 `Scripts` 的箱子裡交給快遞。系統會確保你的箱子被精準地送到舞台後方的道具區，而不會擋到前面的觀眾看表演。

這整個機制分為兩個動作：**設定收件區** 和 **打包寄件**。

---
### 2.2 第一步：在主版面設定「收件區」

打開 `_Layout.cshtml`，滑到最下面、在 `</body>` 標籤的正上方，加上這一行：

```html
<!-- _Layout.cshtml -->
<html>
  <head>
    <title>我的系統</title>
    <!-- 全站共用的 CSS 在這裡 -->
  </head>
  <body>

    @RenderBody()  <!-- 子頁面的主體內容會插入這裡 -->

    <!-- 在底部載入 jQuery 等核心套件 -->
    <script src="~/lib/jquery/jquery.min.js"></script>
    <script src="~/lib/bootstrap/bootstrap.min.js"></script>

    <!-- 👇 在核心套件之後，設定「Scripts 收件區」 -->
    @RenderSection("Scripts", required: false)

  </body>
</html>
```

**關鍵細節：`required: false` 不能省！**

> ⚠️ **常見踩坑**：`required: false` 的意思是「這個區塊是非必填的」。如果你忘了加，只要某個子頁面沒有提供 `@section Scripts` 的內容，系統等不到包裹就會直接拋出例外錯誤，網頁整個當掉。因為不是每個頁面都需要專屬的 JS，所以這個 `false` 幾乎每次都要加。

---
### 2.3 第二步：在子頁面「打包寄件」

當你在某個特定頁面需要寫 jQuery 或自訂 JS 時，用 `@section Scripts { }` 把 `<script>` 標籤包起來：

```html
<!-- Index.cshtml -->
<h2>這是首頁</h2>
<button id="myBtn">點擊我</button>

<!-- 👇 把專屬的 JS 裝箱，系統會送到 Layout 最底部 -->
@section Scripts {
    <script>
        $(() => {
            $('#myBtn').click(function () {
                alert('包裹成功送達並執行！');
            });
        });
    </script>
}
```

**收發的配對靠「名字」**：`@RenderSection("Scripts", ...)` 和 `@section Scripts { }` 中的 `"Scripts"` 這個名字必須完全一致，系統才能精準對接。名字你可以自己取，只要兩邊一樣就行。

---
### 2.4 老實的快遞員：只管送，不管裝什麼

這個機制有一個很重要的特性：**系統不會檢查你箱子裡裝了什麼**。

> 🤔 **動動腦**：如果我在 `@section Scripts { }` 裡面放的不是 `<script>`，而是一個 `<h1>這是標題</h1>`，會怎樣？

答案是：這個 `<h1>` 會被送到 `_Layout.cshtml` 裡呼叫 `@RenderSection("Scripts")` 的那個位置，也就是**整個頁面最最底部**（通常在 Footer 的下方），導致畫面排版看起來壞掉了。

**小結：** `@section` 只是一個「佔位與替換」機制，它老老實實地把箱子裡的東西送到指定位置，你打包什麼，它就在那裡倒出來。

---
## 三、如果不用 `@section`，會發生什麼事？

這是新手最常踩到的地雷，值得單獨拿出來好好說清楚。

假設我們「不聽話」，偷懶直接在子頁面裡寫 JS，不用 `@section` 包起來：

```html
<!-- Index.cshtml（錯誤示範！） -->
<button id="myBtn">測試按鈕</button>

<script>
    $('#myBtn').click(function () {
        alert('按鈕被點擊了！');
    });
</script>
```

這段 `<script>` 最後會被插進頁面的中間（`@RenderBody()` 的位置），而 jQuery 核心檔案還在頁面最底部等著被載入。

瀏覽器由上往下讀到 `$('#myBtn')` 時，會完全看不懂 `$` 是什麼，因為定義 `$` 的 jQuery 還沒被載入。開發者工具的 Console 就會無情地出現這行紅字：

```
Uncaught ReferenceError: $ is not defined
```

用表格對比一下：

|寫法|最終渲染在 HTML 的位置|執行結果|
|---|---|---|
|直接寫 `<script>`|網頁中間（在 jQuery 載入**前**）|❌ 報錯：找不到 `$`|
|用 `@section Scripts` 包起來|網頁底部（在 jQuery 載入**後**）|✅ 順利執行|

> 💡 **核心觀念**：`@section Scripts` 最大的價值，就是確保你的專屬 JS 能排在正確的位置，等 jQuery 或 Bootstrap 等核心套件都準備好後，才開始執行。**程式碼的出場順序，至關重要。**

---
## 四、什麼時候該用 `@section`？什麼時候該抽成獨立 `.js` 檔？

學會了 `@section Scripts` 之後，實務上你會面臨一個選擇題：這段 JS 到底要寫在 `@section` 裡，還是抽成獨立的 `.js` 檔案？

答案取決於兩個維度：**共用性** 與 **是否需要讀取後端 C# 資料**。

### 4.1 抽成獨立的 `.js` 檔案

**適用時機：** 這段邏輯**多個頁面**都會用到時。

**好處：** 瀏覽器下載一次就會快取起來，使用者在不同頁面之間切換時，不需要重複下載，效能較好。

> 🏢 **生活比喻**：就像公司發佈的「SOP 手冊」，印好一本放在桌上，任何部門需要時都可以直接翻閱，不用每次都重新印。

### 4.2 直接寫在 `@section Scripts` 裡

**適用時機：** 這段邏輯**只有這個頁面專屬**，或是你需要**直接讀取後端 C# 變數**。

**最大殺手鐧**：寫在 `.cshtml` 裡的 JS 可以直接跟 C# 的 Razor 語法混用。獨立的 `.js` 檔是純前端檔案，完全看不懂 `@` 符號；但在 `.cshtml` 裡面，你可以無縫存取後端傳來的 `Model` 或 `ViewBag` 資料：

```html
@section Scripts {
    <script>
        // 直接用 @ 把 C# 的 Model 資料塞給前端的 JavaScript 變數
        let currentUserId = '@Model.UserId';
        console.log("現在登入的使用者 ID 是：" + currentUserId);

        // 接下來就可以拿這個 ID 去打 API 或做其他操作...
    </script>
}
```

**決策對比表：**

|判斷情境|獨立 `.js` 檔案|`@section Scripts` 內寫 JS|
|---|---|---|
|**影響範圍**|跨頁面、全站共用|單一頁面專用|
|**瀏覽器快取**|✅ 會快取（效能佳）|❌ 不會獨立快取（隨 HTML 一次性載入）|
|**讀取 C# 變數**|❌ 無法直接讀取|✅ 可以直接用 `@` 讀取（如 `@Model.Id`）|

---
## 五、獨立 `.js` 檔需要後端資料時，怎麼「偷渡」資料？

這個情境在實務上非常常見：JS 邏輯太龐大，必須抽成獨立 `.js` 檔來管理，但同時又需要用到後端的 C# 資料（例如 `UserId`）。

有三種常見的「偷渡」招式：

---
### 招式一：隱藏欄位（Hidden Input）🥷

最古老也最直覺的做法。在 `.cshtml` 放一個隱藏的 HTML 標籤塞值，再讓 `.js` 把值取出來。

**在 `.cshtml` 裡：**

```html
<input type="hidden" id="hiddenUserId" value="@Model.UserId" />
```

**在獨立的 `.js` 檔裡：**

```javascript
let userId = $('#hiddenUserId').val();
console.log("偷渡成功的 ID：" + userId);
```

---
### 招式二：HTML5 的 `data-*` 屬性 🏷️

當資料跟某個特定的 HTML 元素（例如某個按鈕）有直接關聯時，這個方法最乾淨、語意最正確。

**在 `.cshtml` 裡：**

```html
<button id="myActionBtn" data-userid="@Model.UserId">執行動作</button>
```

**在獨立的 `.js` 檔裡：**

```javascript
$('#myActionBtn').click(function () {
    let userId = $(this).data('userid');
    console.log("從按鈕身上拿到的 ID：" + userId);
});
```

---
### 招式三：在 `@section Scripts` 宣告全域變數 🌍

如果你有「一整包」複雜的設定參數要傳給獨立的 JS，可以先在 `.cshtml` 的 `@section Scripts` 裡宣告好變數，再引入獨立的 `.js` 檔。

**在 `.cshtml` 裡：**

```html
@section Scripts {
    <script>
        // 先用 window 物件定義好全域設定
        window.pageConfig = {
            userId: '@Model.UserId',
            role: '@Model.Role'
        };
    </script>
    <!-- 記得把獨立的 js 檔接在宣告後面載入 -->
    <script src="~/js/my-complex-logic.js"></script>
}
```

**在獨立的 `.js` 檔裡：**

```javascript
let currentUserId = window.pageConfig.userId;
```

---
**三種招式的選用指引：**

|招式|什麼時候用？|優點|注意事項|
|---|---|---|---|
|**隱藏欄位**|傳遞 1～2 個簡單的字串或數字|簡單直覺，新手最容易理解|隱藏欄位太多時，HTML 會顯得凌亂|
|**`data-*` 屬性**|資料與特定 HTML 元素有直接關聯|程式碼最乾淨，符合 HTML5 語意|只能把資料綁在實體的 HTML 標籤上|
|**全域變數（`window`）**|需要傳遞一整包物件或陣列格式的設定|可以一次傳遞複雜資料結構|⚠️ 容易污染全域命名空間，要小心變數名稱衝突|

> 💡 **小結**：當 JS 抽成獨立檔案後，它就失去了直接跟 C# 溝通的能力。我們必須透過 HTML（隱藏欄位或 `data` 屬性）或是瀏覽器的全域空間（`window` 物件）來當作傳遞資料的橋樑。

---
## 六、版面配置的兩大核心：`@RenderBody()` 與 `@RenderSection()`

現在讓我們稍微拉高視角，把整個 Layout 機制的運作方式說清楚。

`_Layout.cshtml` 這個主版面，就像是一套**簡報母片**——有固定的公司 Logo、頁首、頁尾。而裡面有兩種不同的「預留位置」：

|語法|角色|說明|
|---|---|---|
|`@RenderBody()`|主要內容文字框|把子頁面（`Index.cshtml`）的**主體 HTML** 塞進來，每個主版面只能有一個|
|`@RenderSection(name, required)`|客製化預留位置|讓子頁面用 `@section` 把**特定內容**送到這裡，可以有多個、各有不同名字|

實際的配置大致如下：

```html
<!-- _Layout.cshtml 的骨架 -->
<html>
  <head>
    <title>企業系統</title>
    <link rel="stylesheet" href="~/css/site.css" />
  </head>
  <body>

    <!-- 👆 頁首導覽列 -->
    <nav>...</nav>

    <!-- 👇 子頁面的主體 HTML 會插在這裡 -->
    @RenderBody()

    <!-- 👇 頁尾 -->
    <footer>...</footer>

    <!-- 在最底部載入核心 JS 套件 -->
    <script src="~/lib/jquery/jquery.min.js"></script>

    <!-- 👇 Scripts 收件區，放在核心套件之後 -->
    @RenderSection("Scripts", required: false)

  </body>
</html>
```

對應的子頁面：

```html
<!-- Index.cshtml -->
@{ Layout = "_Layout"; }  <!-- 指定要套用哪個主版面 -->

<!-- 這裡的 HTML 會跑到 @RenderBody() 的位置 -->
<h2>這是首頁</h2>
<button id="myBtn">點擊我</button>

<!-- 這個區塊會跑到 @RenderSection("Scripts") 的位置 -->
@section Scripts {
    <script>
        $(() => {
            $('#myBtn').click(function () {
                alert('成功！');
            });
        });
    </script>
}
```

> ⚠️ **注意檔名要完全一致**：`@{ Layout = "_Layout"; }` 裡寫的名字，必須跟實際的 `.cshtml` 檔名完全相符（不含副檔名）。如果你的主版面叫 `_CustLayout.cshtml`，就要寫 `_CustLayout`。名字對不上，頁面就會找不到主版面，整個排版都會壞掉。

---
## 七、`_ViewStart.cshtml`：幫全站統一設定預設版面

現在我們來談一個能幫你省下很多重複功夫的檔案。

想像一下，如果系統有 100 個子頁面，每個都要手動寫 `@{ Layout = "_Layout"; }`——不僅累人，萬一哪個頁面手誤打錯字，那個頁面的排版就會直接壞掉。

> 🏢 **生活比喻**：就像公司規定大家穿制服。老闆不用每天跑到每個員工桌邊耳提面命，只要在公司門口的公佈欄貼一張公告（`_ViewStart.cshtml`）寫著「進來本公司的人，預設一律穿這套制服」，大家就自然會遵守了。

### 7.1 `_ViewStart.cshtml` 怎麼運作？

MVC 在準備讀取任何一個子頁面之前，會先去檢查當前資料夾裡有沒有 `_ViewStart.cshtml`。如果有，會**優先**執行裡面的程式碼。

通常會在 `Views` 資料夾的根目錄建立這個檔案，內容非常簡單：

```csharp
@{
    Layout = "_Layout";
}
```

有了這個公告檔案，子頁面就不需要再自己宣告 `@{ Layout = "_Layout"; }` 了，系統會自動幫它套上。

---
## 八、`Layout = null`：幫特定頁面申請「特許證」

`_ViewStart.cshtml` 幫全站統一設好了制服，但實務上一定有例外。

最典型的例子是**登入頁面**。登入頁通常是乾淨的獨立頁面，只有帳號密碼輸入框，完全不需要套用全站共用的頂部導覽列和底部宣告。

這時候，你只需要在 `Login.cshtml` 的最上方加上這一行，就能推翻 `_ViewStart.cshtml` 的預設設定：

```csharp
@{
    Layout = null;
}
```

> 💡 **`null` vs 空字串 `""`**：兩者都能讓頁面不套用任何主版面，但 `null` 在 C# 的語意上更精確，代表「明確地沒有」，建議養成用 `null` 的習慣。

### 8.1 宣告 `Layout = null` 之後，記得補齊 HTML 結構

這是很多人學會 `Layout = null` 之後常忽略的坑。

當你拔掉主版面，原本放在 `_Layout.cshtml` 裡的 `<html>`、`<head>`、CSS 引用、JS 引用，**全部都沒了**。你必須在這個獨立頁面自己補齊完整的 HTML 結構：

```html
<!-- Login.cshtml -->
@{
    Layout = null;
}

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>系統登入</title>
    <!-- 主版面不見了，CSS 要自己引入 -->
    <link rel="stylesheet" href="~/css/login-style.css" />
</head>
<body>
    <div class="login-box">
        <h2>歡迎登入</h2>
        <!-- 登入表單... -->
    </div>

    <!-- 主版面不見了，需要的 JS 也要自己引入 -->
    <script src="~/lib/jquery/jquery.min.js"></script>
</body>
</html>
```

> ⚠️ **常見踩坑**：忘記補 CSS/JS 引用，導致登入頁面完全沒有樣式，或是 jQuery 失效。這是每次使用 `Layout = null` 都必須自我提醒的事。

---
## 九、Layout 機制大拼圖——總整理

走到這裡，我們把整個 MVC 版面配置機制完整地走了一遍。用一張表格來收斂：

|檔案 / 語法|扮演的角色|解決什麼問題|
|---|---|---|
|**`_Layout.cshtml`**|網頁的「共用母片」|避免每個頁面都要重複寫導覽列、頁首、頁尾的 HTML|
|**`@RenderBody()`**|主版面上的「主要文字框」|決定子頁面的主體 HTML 要插在哪裡|
|**`@RenderSection()`**|主版面上的「客製化預留位置」|讓子頁面的特定區塊（如 JS 腳本）能送到指定位置|
|**`@section Scripts { }`**|子頁面的「專屬包裹」|讓子頁面專用的 JS 精準安插到主版面底部，避免載入順序錯誤|
|**`_ViewStart.cshtml`**|全站的「預設公告」|統一設定全站預設套用的 Layout，省去逐頁設定的麻煩|
|**`Layout = null;`**|單一頁面的「特許證」|推翻預設公告，讓特定頁面（如登入頁）維持完全獨立的排版|

---
## 十、Partial View（部分檢視）：可重複使用的 HTML 積木

掌握了 Layout 機制之後，我們再往前走一步，來談另一個你在實際專案裡會頻繁用到的工具：**Partial View（部分檢視）**。

### 10.1 它解決什麼問題？

假設你的系統裡，「首頁」、「會員中心」、「商品頁」都要在右下角顯示同一個「聯絡客服小卡片」。

你的第一個念頭可能是：「把它放進 `_Layout.cshtml` 不就好了？」

這個直覺很合理，但實務上會遇到兩個問題：

1. **`_Layout.cshtml` 會越來越肥**：導覽列、頁尾、客服卡片、廣告橫幅全部塞進去，這個檔案會變成幾千行，難以維護。
2. **缺乏彈性**：如果這個卡片只想出現在「首頁」和「商品頁」，但不想在「會員中心」出現呢？放進主版面就變成全站強迫中獎了。

**Partial View** 就是為了解決這種「跨頁面共用、但不是全站都需要」的局部區塊問題。

### 10.2 Partial View 是什麼？

你可以把它想像成一塊**模組化的小積木**，或是一張**可以到處貼的便利貼**。

我們把客服卡片的 HTML 獨立寫成一個小檔案，命名為 `_ContactCard.cshtml`。哪個頁面需要，就在那裡呼叫它，系統會自動把 HTML 展開。

> 📌 **命名慣例**：不會單獨作為完整頁面顯示的共用元件，檔名前面習慣加上底線 `_`，方便一眼認出它是「小積木」而非獨立頁面。這只是慣例，不是強制規定，但建議遵守。

### 10.3 實際怎麼做？

**第一步：做積木（建立 `_ContactCard.cshtml`）**

```html
<!-- Views/Shared/_ContactCard.cshtml -->
<div class="card">
    <h4>聯絡客服</h4>
    <p>電話：0800-000-000</p>
</div>
```

**第二步：用積木（在需要的頁面呼叫）**

```html
<!-- Index.cshtml -->
<h2>這是首頁</h2>
<p>歡迎來到首頁</p>

<!-- 透過 partial 標籤，把積木安插進來 -->
<partial name="_ContactCard" />
```

就這樣。系統在渲染這個頁面時，會把 `_ContactCard.cshtml` 的內容展開，插入到 `<partial>` 的位置。

---
### 10.4 進階應用：搭配迴圈顯示多筆資料

Partial View 真正強大的地方，是可以結合 C# 的迴圈語法，動態渲染多筆資料。

假設你做了一個「單一商品卡片」的 Partial View，要在商品列表頁顯示 10 個不同的商品，你可以這樣做：

**商品卡片積木（`_ProductCard.cshtml`）：**

```html
@model ProductViewModel  <!-- 宣告這個積木接受什麼型別的資料 -->

<div class="product-card">
    <img src="@Model.ImageUrl" alt="@Model.Name" />
    <h3>@Model.Name</h3>
    <p>NT$ @Model.Price</p>
</div>
```

**商品列表頁（`ProductList.cshtml`）：**

```html
@model List<ProductViewModel>

<h2>商品列表</h2>
<div class="product-grid">
    @foreach (var product in Model)
    {
        <!-- 每次迴圈都呼叫一次積木，並傳入對應的資料 -->
        <partial name="_ProductCard" model="product" />
    }
</div>
```

這樣一來，無論有幾筆商品，你的列表頁都保持乾淨；卡片的 HTML 只需要在 `_ProductCard.cshtml` 維護一次。

---
### 10.5 Partial View vs. `_Layout.cshtml`：我該用哪個？

| 比較項目     | `_Layout.cshtml`（主版面） | Partial View（部分檢視） |
| -------- | --------------------- | ------------------ |
| **扮演角色** | 網頁的主體骨架               | 局部的功能小區塊           |
| **影響範圍** | 預設套用到全站所有頁面           | 只有明確呼叫它的頁面才會出現     |
| **適用情境** | `<head>`、頂部選單、底部版權宣告  | 客服卡片、商品卡、側邊欄工具     |
| **生活比喻** | 房子的樑柱與外牆              | 買來可以隨意擺放的系統櫃       |
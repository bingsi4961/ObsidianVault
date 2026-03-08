---
date: 2026-03-08 11:37
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

## 目錄

- [[#1. 為什麼需要 HtmlAgilityPack？]]
- [[#2. 核心觀念：DOM 樹狀結構]]
- [[#3. 三個標準動作：HAP 的運作全貌]]
- [[#4. 安裝與環境準備]]
- [[#5. 載入 HTML 的兩種方式]]
- [[#6. XPath 定位語法]]（含 6.1～6.10 共十個小節）
- [[#7. 提取資料：InnerText 與 GetAttributeValue]]
- [[#8. 實戰演練：抓取新聞列表]]
- [[#9. 使用 LINQ 精簡程式碼]]
- [[#10. 處理相對網址：Uri 類別]]
- [[#11. 完整實戰程式碼]]
- [[#12. HAP 的限制：靜態 vs 動態網頁]]

---

## 1. 為什麼需要 HtmlAgilityPack？

### 傳統字串處理的痛苦

當我們想要從網站抓取資料（例如：商品價格、新聞標題），手上拿到的是一大串 HTML 原始碼。如果你用字串處理的方式（例如 `Substring` 或 `IndexOf`）去抓資料，程式碼會變得極其脆弱——只要網頁多了一個空白或換行，你的程式就壞了。

### HTML 不等於 XML

有些人會想：「HTML 看起來跟 XML 很像，用 XML 解析器不就好了？」問題在於，現實世界的 HTML 通常很「髒」：工程師可能會漏寫結束標籤 `</div>`，或者標籤嵌套亂七八糟。**一般的 XML 解析器只要遇到一點點格式錯誤就會直接崩潰**，但瀏覽器卻能照樣顯示這些有問題的 HTML。

### HAP 的定位

**HtmlAgilityPack** 就像是一位很有經驗的「讀信專家」。即便 HTML 寫得再亂、再不標準，它都能像瀏覽器一樣，把這堆亂糟糟的文字轉換成一個很有層次的**樹狀結構（DOM）**，讓你輕鬆找到你要的資料。

以下是各種方法的對比：

| 特性       | 正則表達式 (Regex)    | HtmlAgilityPack (HAP) |
| :------- | :--------------- | :-------------------- |
| **解析方式** | 暴力比對字串樣式         | 理解 HTML 的層次結構         |
| **容錯率**  | 極低（HTML 稍微變動就失效） | 極高（能處理不規範的 HTML）      |
| **易讀性**  | 像密碼一樣難懂          | 像路徑一樣直覺（XPath）        |
| **適用情境** | 找簡單的固定格式         | 處理複雜、有嵌套關係的網頁         |

---

## 2. 核心觀念：DOM 樹狀結構

在學習 HAP 的操作之前，先建立一個直覺的比喻，有助於理解後面所有的語法。

想像一下，**HTML 網頁就像是一個巨大的倉庫**：

- **標籤 (`<div>`, `<a>`, `<h1>`)** 就像是倉庫裡的各種貨架和箱子。
- **屬性 (`class`, `id`, `href`)** 就像是箱子上的標籤貼紙，寫著「生鮮類」、「編號 001」。
- **內容 (Text)** 就是箱子裡面裝的實際貨物。

如果沒有 HAP，你就像是在黑暗中拿著手電筒，試圖從倉庫大門口瞄準某個箱子。有了 HAP，它會幫你把整座倉庫的平面圖畫出來，讓你直接說：「我要去『生鮮區』的『第二層貨架』拿那個『貼著紅標籤』的箱子」。

---

## 3. 三個標準動作：HAP 的運作全貌

使用 HAP 爬取資料，所有工作都能被歸納成三個標準動作：

**① 載入 (Load)** 📥：把網頁原始碼讀進來，讓 HAP 把它變成一棵「樹」。

**② 定位 (Select)** 🎯：告訴 HAP 你要找哪個標籤，通常使用 **XPath** 語法來描述路徑。

**③ 提取 (Extract)** ⛏️：拿到標籤後，決定是要拿它的「文字」還是「屬性值」（例如超連結）。

用「點外送」來比喻的話：「載入」就像是派外送員去餐廳領餐；「定位」就像是在整袋餐點中用手指指向你要的那盒白飯；「提取」就是把餐盒打開，把裡面的飯挖出來。

---

## 4. 安裝與環境準備

在開始撰寫程式前，請先安裝 HtmlAgilityPack 套件。

**方法一：NuGet 套件管理員**

在 Visual Studio 中，開啟「NuGet 套件管理員」，搜尋 `HtmlAgilityPack` 並點擊安裝。

**方法二：終端機指令**

```bash
dotnet add package HtmlAgilityPack
```

安裝完成後，在程式碼最上方加入命名空間引用：

```csharp
using HtmlAgilityPack;
```

---

## 5. 載入 HTML 的兩種方式

HAP 提供兩種「開箱方式」，差別在於你是要 HAP 幫你去**抓網頁**，還是你手上已經有**網頁內容**了。

### 方式一：`HtmlWeb`（適合直接從網址抓取）

這就像是**叫外送**。你只要給地址（URL），`HtmlWeb` 就會負責幫你跑腿，把網頁抓回來並解析好。

```csharp
// 建立一個「外送員」物件，負責去網路上下載 HTML
var web = new HtmlWeb();

// 告訴外送員去哪個網址領取網頁（載入並解析）
var doc = web.Load("https://example.com");
```

- **優點**：方便，一行程式碼搞定連網與解析。
- **缺點**：控制力較低，無法自訂複雜的連線設定（例如特殊的 Header 或 Cookie）。

### 方式二：`HtmlDocument.LoadHtml()`（適合手上已有 HTML 字串）

這就像是**自己下廚**。你手上已經有食材（HTML 字串）了，只是拿盤子（`HtmlDocument`）把它裝進去整理好。

```csharp
// 假設你已經從某個地方（例如 HttpClient）取得了 HTML 字串
string htmlString = "<div><h1>哈囉！HAP</h1></div>";

// 建立一個空的文件物件
var doc = new HtmlDocument();

// 把 HTML 字串載入進去解析
doc.LoadHtml(htmlString);
```

- **優點**：靈活性極高。如果你是用 `HttpClient` 抓取網頁（可自訂 Header、Cookie 等），就必須用這個方法。
- **缺點**：你需要自己先處理好「如何取得 HTML 字串」這件事。

### 兩種方式的對比

|工具|輸入內容|是否包含網路功能|適用情境|
|:--|:--|:--|:--|
|**`HtmlWeb`**|URL（網址）|✅ 是，它會幫你連上網|直接抓取公開網頁|
|**`HtmlDocument`**|String（字串）|❌ 否，它只負責解析|資料已存在記憶體、檔案或 API 回傳中|

> **【補充說明】** 在實務開發中，許多資深工程師更偏好使用 `HttpClient` 搭配 `doc.LoadHtml()` 的組合。`HttpClient` 是 .NET 官方最強大的網路工具，可以精細控制 Cookie、代理伺服器（Proxy）和超時時間。先把 HTML 抓成字串，再餵給 HAP 解析，這種「分工合作」的方式在處理需要登入或複雜 Header 的網頁時會更穩定。

---

## 6. XPath 定位語法

不論用哪種方式載入，後續的「定位」步驟都是一樣的——透過 **XPath** 語法告訴 HAP 你要找哪個節點。XPath 全名是 **XML Path Language**，你可以把它想像成是網頁地圖中的 **GPS 定位系統**：HTML 網頁是一棟充滿房間的大樓，XPath 就是精確的指引，告訴程式「去 3 樓左手邊第二個房間，拿走那個紅色的盒子」。

以下是 XPath 最常用的四個核心符號：

|符號|名稱|說明|比喻|
|:--|:--|:--|:--|
|`/`|單斜線|找「直接下一層」的子節點|親生孩子|
|`//`|雙斜線|在整份文件中尋找，不論層級多深|所有的後代子孫|
|`[]`|中括號|加入篩選條件（稱為「謂詞」）|穿紅衣服的孩子|
|`@`|艾特符號|選取標籤內的屬性（如 `id`、`class`）|孩子的身分證字號|

### 6.1 基本標籤與屬性定位

最基礎的用法是直接寫標籤名稱，再配合 `[]` 和 `@` 縮小範圍。

```xpath
//h1                        → 找出頁面上所有的 h1 標籤
//div[@id='main']           → 找 id 為 "main" 的 div（id 通常是唯一的）
//a[@class='link']          → 找 class 為 "link" 的超連結
```

用廣播來比喻的話：`//p` 像是「請所有**姓段落**的人過來」；`//p[@id='intro']` 則是「請那位**姓段落**且**身分證號碼是 intro** 的人過來」，指向性精準許多。

### 6.2 `contains()` 函數：處理多個 class 的陷阱

在現代網頁中，一個標籤往往同時擁有多個 class，例如：

```html
<a class="buy-btn primary active">立即購買</a>
```

如果你直接寫 `//a[@class='buy-btn']`，XPath 因為**不是完全匹配**整個 class 字串而找不到它。這時就需要 `contains()` 函數：

```xpath
//a[contains(@class, 'buy-btn')]
```

這行的意思是：「找所有的 `a` 標籤，只要它的 class **包含** `buy-btn` 這幾個字就算符合」。`contains()` 的語法結構是 `contains(@屬性, '要找的部分文字')`，它解決了完整匹配的限制，是爬蟲實務中非常高頻使用的工具。

### 6.3 `/` 與 `//` 的差異，以及 `//` 的本質

這是 XPath 中最容易讓人混淆的地方，值得仔細理解。

`/` 代表「直接子節點（Child）」，只看目標的第一層；`//` 代表「後代節點（Descendant）」，會搜尋目標下方的任何層級。用打開箱子來比喻：`/` 是打開大箱子只看最上面的東西；`//` 是不但打開大箱子，還把裡面所有的小盒子、氣泡紙全部拆開，直到翻出你要的東西為止。

|結構|`/a` 能抓到嗎？|`//a` 能抓到嗎？|
|:--|:--|:--|
|`<div><a>連結</a></div>`|✅ 可以|✅ 可以|
|`<div><span><a>連結</a></span></div>`|❌ 不行（被 span 包住了）|✅ 可以|

進一步問：`//` 的本質是什麼？在 XPath 的完整語法中，`//` 實際上是 `/descendant-or-self::node()/` 的縮寫。這意味著：

- **單斜線 `/`** 對應的完整寫法是 `/child::`，意思是「往下一層」。
- **雙斜線 `//`** 對應的是「從目前位置出發，往下的所有層級（包含自己、兒子、孫子、曾孫……）」。

關鍵在於：**`/` 本身並不代表方向，它是一個「步驟分隔符」**，可以想成 GPS 指令之間的「然後」。以 `//div[@id='news-section']/following-sibling::div` 為例，它的邏輯是：「先找到這個 div（第一步），**然後**（`/`）沿著兄弟軸往後找（第二步）」。`/` 後面接的軸（Axis）決定了移動的方向，不是 `/` 本身決定方向：

- 接 `child::`（預設，可省略）→ 往下一層找
- 接 `following-sibling::` → 往同一層的後面找
- 接 `parent::` → 往上一層找

### 6.4 用屬性縮小搜尋範圍：`[@屬性名="值"]`

只有 `//` 和標籤名稱還不夠精準，現實網頁上通常有很多相同的標籤。我們可以利用 `[@屬性名="值"]` 指定一個更精確的「門牌號碼」：

```xpath
//section[@id='hot-news']      → 找 id 為 "hot-news" 的 section
//img[@class='user-photo']     → 找 class 為 "user-photo" 的 img
//div[@class='item']           → 找 class 為 "item" 的 div
```

舉例說明，假設 HTML 結構如下：

```html
<section id="hot-news">
    <div class="wrapper">
        <div class="item">新聞內容</div>
    </div>
</section>
```

- `//section[@id='hot-news']/div[@class='item']`（單斜線）：找不到 `item`，因為 `item` 是 `wrapper` 的子節點，是 `section` 的「孫子」而不是「兒子」。
- `//section[@id='hot-news']//div[@class='item']`（雙斜線）：可以找到 `item`，因為 `//` 會翻遍 `section` 內部的所有層級。

在爬蟲實務中，`//` 是更常用的選擇，因為前端工程師可能隨時多加一層包裝用的 `div`，使用 `//` 可以讓程式碼更有彈性、不容易因網頁結構微調而失效。

### 6.5 相對路徑：`.` 的重要性

這是新手最常踩的坑！當你已經取得了一個父節點（例如某則新聞的 `div`），想從這個節點「往內」繼續搜尋時，XPath 開頭必須加一個**點（`.`）**：

```csharp
// ❌ 錯誤：會從整份文件的開頭重新找，可能找到其他地方的 <a> 標籤
var aNode = newsNode.SelectSingleNode("//a");

// ✅ 正確：只在 newsNode 內部尋找 <a> 標籤
var aNode = newsNode.SelectSingleNode(".//a");
```

`.//a` 是「只在目前這個房間找」，`//a` 是「去整棟大樓任何地方找」。如果你忘了加 `.`，不管跑幾次迴圈，每次抓到的都會是整份網頁最開頭的第一個連結。

### 6.6 根據文字內容定位：`text()` 與 `.`

有時候，標籤沒有 `id` 也沒有獨特的 `class`，但它顯示的**文字**是固定的，這時可以用 `text()` 函數來定位：

```xpath
//a[text()='點我閱讀']              → 精確匹配，文字剛好就是「點我閱讀」
//a[contains(text(), '閱讀')]       → 部分匹配，文字中包含「閱讀」二字
//button[contains(text(), '提交')]  → 找文字包含「提交」的按鈕
```

不過這裡有個實務細節要注意：如果文字被包在更深層的子標籤裡，`text()` 會失效：

```html
<button><span>提交</span>表單</button>
```

- `//button[contains(text(), '提交')]`：`text()` 只看 `button` 直接夾住的文字。在這個例子中，「提交」在 `span` 裡面，`text()` 看不到，定位失敗。❌
- `//button[contains(., '提交')]`：這裡的「`.`」代表該節點的所有**字串值（String-value）**，會把所有子孫標籤的文字揉成一團來檢查，因此能成功找到。✅

**結論**：當你要根據文字定位時，優先使用 `contains(., '關鍵字')` 而非 `contains(text(), '關鍵字')`，前者更能應對複雜的巢狀結構。

### 6.7 組合多重條件：`and` 與 `or`

當單一條件不夠精準時（例如頁面上有 10 個 `class="btn"` 的按鈕），可以用 `and` 或 `or` 組合多個篩選條件：

```xpath
//button[@class='btn-blue' and text()='確認']
→ 找 class 是 btn-blue，且文字剛好是「確認」的按鈕（兩個條件都要符合）

//input[@type='text' or @type='email']
→ 找 type 是 text 或 email 的輸入框（符合其中一個即可）
```

兩者的語法結構都是在中括號 `[]` 內，用 `and` 或 `or` 串接各個條件。

### 6.8 索引定位：`[n]` 與 `last()`

有時候，頁面上一整排清單的標籤屬性完全相同，這時可以用「順序」來點名：

```xpath
//li[1]           → 選取第一個 li（注意：XPath 索引從 1 開始，不是 0）
//li[last()]      → 選取最後一個 li
//li[last()-1]    → 選取倒數第二個 li
//li[position()<3]→ 選取前兩個 li
```

`last()` 是一個內建函數，代表目前這組結果的總數量（也就是最後一個的編號）。它**不接受參數**，不能寫成 `last(2)`。如果要表達「倒數第幾個」，要用減法：`last() - 1` 就是倒數第二個，`last() - 2` 就是倒數第三個，依此類推。

此外，`//li[1]` 有時會抓到「每個清單裡的第一個」。如果要確保只取整份文件裡的第一個，可以用括號包起來：`(//li)[1]`。

### 6.9 軸導航（Axes）：用鄰居找目標

這是 XPath 最強大的招式：**當目標標籤本身沒有任何特徵時，透過它的「鄰居」來定位**。

常見的軸（Axis）有以下幾種：

|軸名稱|方向|說明|
|:--|:--|:--|
|`following-sibling::`|同一層，往後|找後面的兄弟節點|
|`preceding-sibling::`|同一層，往前|找前面的兄弟節點|
|`parent::`|往上一層|找父節點|
|`..`|往上一層|`parent::` 的縮寫|

以下面的 HTML 為例：

```html
<div class="row">
    <span class="label">價格：</span>
    <span class="value">999</span>
</div>
```

如果 `class="value"` 在頁面上到處都是，但「價格：」這個文字是固定的，你可以這樣說：「先找到文字是『價格：』的標籤，然後找它右邊的鄰居」：

```xpath
//span[text()='價格：']/following-sibling::span
```

這行指令拆解如下：先找到文字為「價格：」的 `span`（步驟一），然後（`/`）沿著兄弟軸往後找同層的 `span`（步驟二），就能精準取到 `999`。

再舉一個表格的例子：

```html
<table id="inventory">
    <tr><td>蘋果</td><td>$50</td></tr>
    <tr><td>香蕉</td><td>$30</td></tr>
</table>
```

要抓「香蕉」旁邊的 `$30`：

```xpath
//td[text()='香蕉']/following-sibling::td
```

注意軸的名稱是**單數** `following-sibling`，不是 `following-siblings`（沒有 s）。

### 6.10 SelectSingleNode vs SelectNodes

HAP 提供兩個主要的定位方法，對應兩種不同的需求：

- **`SelectSingleNode`**：在整份文件（或指定節點）中，**只回傳第一個**符合條件的節點，型別為 `HtmlNode`。
- **`SelectNodes`**：把**所有**符合條件的節點都回傳，型別為 `HtmlNodeCollection`（可用 `foreach` 遍歷）。

```csharp
// 只拿第一個 h1 標籤
var h1Node = doc.DocumentNode.SelectSingleNode("//h1");

// 拿所有的 div.item 標籤（清單）
var allItems = doc.DocumentNode.SelectNodes("//div[@class='item']");
```

---

## 7. 提取資料：InnerText 與 GetAttributeValue

定位到節點之後，下一步是把裡面的資料「挖出來」。根據你要拿的東西不同，有兩種主要的方式。

### 7.1 拿文字內容：`InnerText`

取得標籤「內部」的所有文字。

```csharp
var pNode = doc.DocumentNode.SelectSingleNode("//p");
string text = pNode.InnerText; // 取得 <p> 裡的文字
```

⚠️ **注意**：`InnerText` 會抓到該節點**內部所有子標籤的文字**，包含空白與換行。因此建議搭配 `.Trim()` 去除多餘空白，並盡量針對最精確的標籤（例如 `<p>`）提取，而不是針對父層的 `<div>` 提取：

```csharp
// 建議：針對 <p> 取文字，並去除空白
string title = pNode?.InnerText.Trim() ?? "無標題";
```

### 7.2 拿屬性值：`GetAttributeValue()`

取得標籤上的某個屬性值，例如連結的 `href`、圖片的 `src`。

```csharp
var aNode = doc.DocumentNode.SelectSingleNode("//a");

// 取得連結文字
string text = aNode.InnerText;

// 取得連結網址（第二個參數是找不到時的「預設值」）
string url = aNode.GetAttributeValue("href", "未找到網址");
```

`GetAttributeValue` 的第二個參數是「預設值」，這是非常安全的寫法。如果這個 `<a>` 標籤剛好沒有寫 `href`，程式不會崩潰，而是會回傳你設定的預設值。

用同樣的邏輯，你可以取得圖片路徑：

```csharp
var imgNode = doc.DocumentNode.SelectSingleNode("//img");
string imgSrc = imgNode.GetAttributeValue("src", "未找到圖片路徑");
```

### 7.3 空值防護：`?.` 運算子

爬蟲的世界充滿不確定性——萬一某則新聞剛好沒有圖片，`SelectSingleNode` 就會回傳 `null`。如果直接在 `null` 上呼叫方法，程式會拋出 `NullReferenceException` 並崩潰。

有兩種防護寫法：

**寫法一：if 判斷**

```csharp
var imgNode = node.SelectSingleNode(".//img");
if (imgNode != null)
{
    string imgSrc = imgNode.GetAttributeValue("src", "");
}
```

**寫法二：`?.` 運算子（更簡潔）**

```csharp
// 如果 imgNode 是 null，則整個表達式直接回傳 null，不會拋出例外
string imgSrc = imgNode?.GetAttributeValue("src", "");
```

在爬蟲程式中，`?.` 運算子是讓程式更強韌（Robust）的重要習慣。

---

## 8. 實戰演練：抓取新聞列表

現在我們把前幾章的所有觀念組合起來，完成一個完整的實戰任務。

### 練習用的 HTML 結構

```html
<!DOCTYPE html>
<html>
<body>
    <h1>我的新聞網</h1>

    <section id="hot-news">
        <h2>🔥 熱門頭條</h2>
        
        <div class="item">
            <a href="https://news.example.com/tech-001">
                <img src="images/tech.jpg" alt="科技新聞縮圖">
                <p>2026 年最強 AI 晶片問世，效能提升 200%</p>
            </a>
        </div>

        <div class="item">
            <a href="https://news.example.com/life-002">
                <img src="images/coffee.jpg" alt="生活新聞縮圖">
                <p>研究發現：每天喝一杯咖啡有助於邏輯思考</p>
            </a>
        </div>
    </section>

    <section id="old-news">
        <h2>📜 歷史存檔</h2>
        <div class="item">
            <a href="https://news.example.com/old-999">
                <p>去年的天氣報告</p>
            </a>
        </div>
    </section>
</body>
</html>
```

注意 HTML 中有一個「干擾項」：`id="old-news"` 的 section 裡也有 `class="item"` 的 div，但我們只想要抓熱門新聞區的資料。

### 定義資料模型

首先，定義一個 C# 類別來存放每一則新聞的資料：

```csharp
public class News
{
    public string Title { get; set; }
    public string Url { get; set; }
    public string ImageUrl { get; set; }
}
```

### 抓取邏輯：分兩層定位

針對這種「有干擾項」的情境，最佳策略是「先縮小範圍，再精確打擊」：

```csharp
var newsList = new List<News>();

// 第一層：先定位到「熱門新聞」這個大箱子，排除舊聞區
var hotNode = doc.DocumentNode.SelectSingleNode("//section[@id='hot-news']");

if (hotNode != null)
{
    // 第二層：在大箱子內部，找出所有的「item」小箱子
    // 注意：用 .// 表示「只在 hotNode 內部找」
    var items = hotNode.SelectNodes(".//div[@class='item']");

    if (items != null)
    {
        foreach (var node in items)
        {
            // 定位並提取各項資料，全程使用 .// 相對路徑
            var pNode = node.SelectSingleNode(".//p");
            string title = pNode?.InnerText.Trim() ?? "無標題";

            var aNode = node.SelectSingleNode(".//a");
            string url = aNode?.GetAttributeValue("href", "未找到 Url");

            var imgNode = node.SelectSingleNode(".//img");
            string imageUrl = imgNode?.GetAttributeValue("src", "未找到 Image Url");

            // 建立 News 物件並加入清單
            newsList.Add(new News
            {
                Title = title,
                Url = url,
                ImageUrl = imageUrl
            });
        }
    }
}
```

### 為什麼在迴圈內要針對 `node` 而非 `doc.DocumentNode` 搜尋？

這是爬蟲邏輯中最重要的觀念之一。在 `foreach` 迴圈裡，`node` 代表的是「目前這則新聞的節點」。如果你在迴圈內改用 `doc.DocumentNode` 去搜尋，程式每次都會跳回「整份文件」的最開頭重新找，結果不管跑幾次迴圈，抓到的永遠都是網頁上的第一個連結。

---

## 9. 使用 LINQ 精簡程式碼

當你對爬蟲邏輯熟悉之後，可以使用 **LINQ** 把冗長的 `foreach` 改寫成更簡潔的流式語法。LINQ 就像是一條「自動化生產線」，把「定位→提取→建立物件→加入清單」這四個步驟串成一個流暢的動作。

### LINQ 改寫版本

```csharp
var newsList = doc.DocumentNode
    // 直接用一條 XPath 定位所有目標節點
    .SelectNodes("//section[@id='hot-news']//div[@class='item']")
    // 用 ?. 防止 SelectNodes 回傳 null 時崩潰
    ?.Select(node => new News
    {
        Title    = node.SelectSingleNode(".//p")?.InnerText.Trim() ?? "無標題",
        Url      = node.SelectSingleNode(".//a")?.GetAttributeValue("href", "未找到 Url"),
        ImageUrl = node.SelectSingleNode(".//img")?.GetAttributeValue("src", "未找到 Image Url")
    })
    .ToList()
    // 如果 SelectNodes 回傳 null，就給一個空清單而非 null
    ?? new List<News>();
```

### 傳統 `foreach` vs LINQ 的比較

|特性|傳統 `foreach`|LINQ 寫法|
|:--|:--|:--|
|**程式碼長度**|較長，需要處理中間變數|極短，一氣呵成|
|**可讀性**|重點分散在各行|專注於「轉換」的邏輯|
|**後續過濾**|要過濾資料得再加 `if`|直接串接 `.Where()` 即可|

### LINQ 的後續資料處理

取得 `newsList` 後，可以繼續用 LINQ 進行過濾與排序：

```csharp
// 只看標題含有「AI」的新聞，並依標題長度排序
var filteredNews = newsList
    .Where(news => news.Title.Contains("AI"))
    .OrderBy(news => news.Title.Length) // 字串長度用 .Length，不是 .Count()
    .ToList();
```

> **注意**：取得字串長度應使用 `.Length` 屬性，而非 `.Count()`。`.Length` 是字串（`string`）的屬性；`.Count()` 是 LINQ 提供給集合（`IEnumerable`）的方法。

---

## 10. 處理相對網址：Uri 類別

這是爬蟲實務中「百分之百會遇到」的問題。有時候網頁上的連結並不是完整的 `https://...`，而是相對路徑：

```html
<a href="/news/123">某則新聞</a>
<img src="images/photo.jpg">
```

如果直接把 `/news/123` 存起來，點擊時會發現它根本連不到任何地方，必須補全成 `https://news.example.com/news/123`。

### 為什麼不用 `Path.Combine`？

雖然 `Path.Combine` 在處理本機「檔案路徑」時很好用，但它並不適合處理網址：

- **斜線方向不同**：在 Windows 系統上，`Path.Combine` 可能產生反斜線 `\`，而網址規定永遠使用正斜線 `/`。
- **根目錄處理邏輯不同**：`Path.Combine` 不理解 `https://` 這種通訊協定前綴。

### 正確做法：使用 `Uri` 類別

.NET 提供了專門處理網址的 **`Uri`** 類別，它能自動處理斜線、協定前綴等複雜問題：

```csharp
string baseUrl = "https://news.example.com";

// 寫法一：兩個字串參數（baseUrl 必須是絕對路徑）
Uri fullUri1 = new Uri(new Uri(baseUrl), "/news/123");
Console.WriteLine(fullUri1.ToString());
// 輸出：https://news.example.com/news/123

// 寫法二：先建立 Uri 物件，再用它來拼接（更安全，推薦）
Uri baseUri = new Uri(baseUrl);
Uri fullUri2 = new Uri(baseUri, "images/photo.jpg");
Console.WriteLine(fullUri2.ToString());
// 輸出：https://news.example.com/images/photo.jpg
```

`Uri` 的優點是：不論 `baseUrl` 結尾有沒有 `/`，或者相對路徑開頭有沒有 `/`，它都能自動校正，不會產生 `//` 重複的問題。

### 在迴圈中整合 Uri

```csharp
string baseUrl = "https://news.example.com";

foreach (var node in items)
{
    var aNode = node.SelectSingleNode(".//a");
    string relativeUrl = aNode?.GetAttributeValue("href", "");

    var imgNode = node.SelectSingleNode(".//img");
    string relativeImgPath = imgNode?.GetAttributeValue("src", "");

    var news = new News
    {
        // 使用三元運算子：有值才拼接，沒有值就給預設文字
        Url = !string.IsNullOrEmpty(relativeUrl)
                ? new Uri(new Uri(baseUrl), relativeUrl).ToString()
                : "未找到 Url",

        ImageUrl = !string.IsNullOrEmpty(relativeImgPath)
                ? new Uri(new Uri(baseUrl), relativeImgPath).ToString()
                : "未找到 Image Url"
    };

    newsList.Add(news);
}
```

---

## 11. 完整實戰程式碼

將本文所有觀念整合，以下是一個完整可執行的範例：

```csharp
using HtmlAgilityPack;
using System;
using System.Collections.Generic;
using System.Linq;

// ── 資料模型 ──────────────────────────────────────────────────────────────
public class News
{
    public string Title    { get; set; }
    public string Url      { get; set; }
    public string ImageUrl { get; set; }
}

// ── 主程式 ────────────────────────────────────────────────────────────────
class Program
{
    static void Main(string[] args)
    {
        string baseUrl = "https://news.example.com";

        // ① 載入：讓 HtmlWeb 去網路上抓取網頁並解析
        var web = new HtmlWeb();
        var doc = web.Load(baseUrl);

        // ② 定位與提取（LINQ 版本）
        var newsList = doc.DocumentNode
            .SelectNodes("//section[@id='hot-news']//div[@class='item']")
            ?.Select(node =>
            {
                // 定位各子節點
                var aNode   = node.SelectSingleNode(".//a");
                var pNode   = node.SelectSingleNode(".//p");
                var imgNode = node.SelectSingleNode(".//img");

                // 提取相對路徑
                string relativeUrl = aNode?.GetAttributeValue("href", "") ?? "";
                string relativeImg = imgNode?.GetAttributeValue("src", "") ?? "";

                // 拼接成完整網址
                string fullUrl = !string.IsNullOrEmpty(relativeUrl)
                    ? new Uri(new Uri(baseUrl), relativeUrl).ToString()
                    : "未找到 Url";

                string fullImg = !string.IsNullOrEmpty(relativeImg)
                    ? new Uri(new Uri(baseUrl), relativeImg).ToString()
                    : "未找到 Image Url";

                return new News
                {
                    Title    = pNode?.InnerText.Trim() ?? "無標題",
                    Url      = fullUrl,
                    ImageUrl = fullImg
                };
            })
            .ToList() ?? new List<News>();

        // ③ 輸出結果
        Console.WriteLine($"共抓到 {newsList.Count} 則新聞：");
        foreach (var news in newsList)
        {
            Console.WriteLine($"標題：{news.Title}");
            Console.WriteLine($"連結：{news.Url}");
            Console.WriteLine($"圖片：{news.ImageUrl}");
            Console.WriteLine("---");
        }

        // ④ 後續資料處理：過濾 + 排序
        var aiNews = newsList
            .Where(n => n.Title.Contains("AI"))
            .OrderBy(n => n.Title.Length)
            .ToList();

        Console.WriteLine($"\n含有「AI」關鍵字的新聞共 {aiNews.Count} 則。");
    }
}
```

---

## 12. HAP 的限制：靜態 vs 動態網頁

學完本文的內容，你已經能應付大多數「靜態網頁」的抓取任務了。但在實務中，還需要了解 HAP 的一個重要限制。

### 什麼是靜態網頁？

`HtmlWeb.Load()` 和 `doc.LoadHtml()` 拿到的，都是伺服器「第一時間回傳」的 HTML 原始碼。如果網頁的內容就在這份 HTML 裡，我們就稱之為**靜態網頁**，HAP 可以完美處理。

### 什麼是動態網頁？

有些網站的資料是靠 **JavaScript** 在瀏覽器端執行後才產生的（例如：捲動才會出現的社群媒體貼文、需要點擊按鈕才顯示的資料）。**HAP 預設只能抓到「第一時間下載下來的 HTML」**，對於動態產生的內容，HAP 看到的會是一片空白。

如果你在瀏覽器看得到資料，但 HAP 抓下來卻是空的，十之八九就是碰到了動態網頁的問題，這時就需要使用 Selenium 等能夠驅動真實瀏覽器的工具來解決。

---

## 學習成果總結

完成本篇教學後，你已經掌握了以下能力：

1. **解釋 HAP 的價值**：了解為什麼 HAP 比正則表達式和 XML 解析器更適合處理真實世界的 HTML。
2. **載入網頁**：會用 `HtmlWeb` 和 `HtmlDocument.LoadHtml()` 兩種方式載入 HTML。
3. **XPath 基礎定位**：理解 `//`（後代）與 `/`（子代）的差異，以及 `.//`（相對路徑）的正確用法。
4. **XPath 進階篩選**：能用 `[@id='...']`、`[@class='...']`、`contains()` 縮小搜尋範圍，並用 `text()` 與 `.` 根據文字內容定位。
5. **XPath 組合條件**：能用 `and` / `or` 組合多個篩選條件，精準鎖定大眾臉的標籤。
6. **XPath 索引定位**：能用 `[n]`、`[last()]`、`[last()-n]` 指定清單中的特定項目。
7. **XPath 軸導航**：能用 `following-sibling::`、`preceding-sibling::`、`parent::` 透過鄰居節點找到沒有特徵的目標。
8. **資料提取**：會用 `InnerText` 取文字，`GetAttributeValue` 取屬性值，並搭配 `?.` 防止 `null` 造成崩潰。
9. **整合 LINQ**：能把爬蟲邏輯精簡成流式的 LINQ 語法，並用 `.Where()` 和 `.OrderBy()` 進行後續資料處理。
10. **處理相對網址**：了解為何不能用 `Path.Combine` 處理 URL，並正確使用 `Uri` 類別拼接完整網址。
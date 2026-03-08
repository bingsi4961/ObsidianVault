---
date: 2026-03-08 11:37
aliases:
tags:
  - CSharp
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[HtmlAgilityPack 教學（二）：XPath 定位語法完全指南]]
#### 📑 [[HtmlAgilityPack 教學（三）：實戰整合篇 — 從 HTML 到 C_Sharp 物件]]

---

## 為什麼要學 HtmlAgilityPack？它解決什麼問題？🤔

在實務上，當我們想要從網站抓取資料（例如：抓取電商網站的商品價格、新聞標題）時，我們會拿到一堆 **HTML 原始碼**。問題是——這堆原始碼該怎麼處理？

### 傳統方式的痛苦 😫

如果你用處理字串的方式（例如 `Substring` 或 `IndexOf`）去抓資料，程式碼會變得極其脆弱。只要網頁多了一個空白或換行，你的程式就壞了。

### HTML 不是 XML 🚫

有些人會想：「HTML 看起來跟 XML 很像，我用 XML 解析器不就好了？」

問題在於，現實世界的 HTML 通常很「髒」。工程師可能會漏寫結束標籤 `</div>`，或者標籤嵌套亂七八糟。**一般的 XML 解析器只要遇到一點點格式錯誤就會直接崩潰**，但瀏覽器卻能照樣顯示。

### HAP 的定位：包容力強大的讀信專家 ✉️

**HtmlAgilityPack（簡稱 HAP）** 就像是一位很有經驗的「讀信專家」。即便 HTML 寫得再亂、再不標準，它都能像瀏覽器一樣，把這堆亂糟糟的文字轉換成一個很有層次的**樹狀結構（DOM）**，讓你像在翻閱資料夾一樣，輕鬆找到你要的那份文件。

### 與正則表達式的對比 ⚖️

|特性|正則表達式 (Regex)|HtmlAgilityPack (HAP)|
|---|---|---|
|**解析方式**|暴力比對字串樣式|理解 HTML 的層次結構|
|**容錯率**|極低（HTML 稍微變動就失效）|極高（能處理不規範的 HTML）|
|**易讀性**|像密碼一樣難懂|像路徑一樣直覺 (XPath)|
|**適用情境**|找簡單的固定格式|處理複雜、有嵌套關係的網頁|

---

## 日常生活比喻：在亂糟糟的倉庫找貨物 📦

想像一下，HTML 網頁就像是一個**巨大的倉庫**：

- **標籤 (`<div>`, `<a>`, `<h1>`)**：就像是倉庫裡的各種貨架和箱子。
- **屬性 (`class`, `id`, `href`)**：就像是箱子上的標籤貼紙，寫著「生鮮類」、「編號 001」。
- **內容 (Text)**：就是箱子裡面裝的實際貨物。

如果沒有 HAP，你就像是在黑暗中拿著手電筒，試圖從倉庫大門口直接瞄準某個箱子。有了 **HAP**，它會幫你把整座倉庫的平面圖畫出來，讓你直接說：「我要去『生鮮區』的『第二層貨架』拿那個『貼著紅標籤』的箱子」。

---

## 學習地圖：HAP 的運作全貌 🗺️

在開始寫程式碼之前，我們先看清楚這塊拼圖長什麼樣子。使用 HAP 基本上只有三個標準動作：

1. **載入 (Load)** 📥：把網頁原始碼讀進來，讓 HAP 把它變成一棵「樹」。
2. **定位 (Select)** 🎯：告訴 HAP 你要找哪個標籤（通常使用 **XPath** 或 **CSS Selector**）。
3. **提取 (Extract)** ⛏️：拿到標籤後，決定是要拿它的「文字」還是「屬性值」（例如連結）。

接下來，我們就從最簡單的實戰範例開始，讓你親手操作一次這三個步驟。

---

## 環境準備 🔧

在開始寫程式碼之前，請先確保你在專案中安裝了 **HtmlAgilityPack**。

如果你使用 Visual Studio，請在「NuGet 套件管理員」搜尋 `HtmlAgilityPack` 並安裝；或者在終端機輸入：

```bash
dotnet add package HtmlAgilityPack
```

---

## 第一個實戰範例：抓取網頁標題 🚀

請將以下程式碼貼進你的 `Main` 方法中：

```csharp
using HtmlAgilityPack;
using System;

// 1. 建立一個「外送員」物件，負責去網路上下載 HTML
var web = new HtmlWeb();

// 2. 告訴外送員去哪間餐廳領餐 (載入網頁)
var doc = web.Load("https://example.com");

// 3. 在整份菜單中，找到標籤叫做 <h1> 的那一行 (定位)
// "//h1" 的意思是：不管在哪個層級，幫我找到標籤名稱是 h1 的節點
var h1Node = doc.DocumentNode.SelectSingleNode("//h1");

// 4. 如果找到了，就把裡面的文字印出來 (提取)
if (h1Node != null)
{
    Console.WriteLine("找到的大標題是：" + h1Node.InnerText);
}
else
{
    Console.WriteLine("找不到標籤...");
}
```

### 💡 這裡發生了什麼事？

這段程式碼展示了 HAP 的核心邏輯——載入、定位、提取。讓我們用生活化的方式來理解：

- **載入** 📥：就像派一個外送員（`HtmlWeb`）去餐廳把餐點（HTML 原始碼）領回來。
- **定位** 🎯：在整袋餐點中，用手指指向你要的那盒「白飯」（`<h1>` 標籤）。這裡使用了 `//h1` 這個叫做 **XPath** 的語法，你可以把它想像成電腦裡的「檔案路徑」。`//` 代表「在整份文件中搜尋」，`h1` 代表你要找的「標籤名稱」。
- **提取** ⛏️：把餐盒打開，把裡面的「飯」（文字內容）挖出來。

> ⚠️ **老師的踩坑筆記**：小白最常遇到的問題
> 
> 在實務中，新手最常卡住的地方不是寫不出程式，而是**「抓不到東西」**。這通常有兩個原因：
> 
> 1. **節點不存在**：網頁結構變了，或者你選錯了標籤。所以程式碼裡一定要寫 `if (node != null)`，不然程式會直接閃退（拋出 `NullReferenceException`）。
> 2. **動態內容**：有些網頁的資料是靠 JavaScript 後來才產生的（例如：捲動才會出現的臉書貼文），HAP 預設只能抓到「第一時間下載下來的 HTML」。如果資料是動態產生的，HAP 看到的會是一片空白。

---

## 認識 `SelectSingleNode` 與 `SelectNodes` 📋

通常一個網頁不會只有一個段落或一個連結。HAP 提供了兩個方法來應對不同的需求：

- **`SelectSingleNode`**：在文件中**只會回傳它第一個找到的標籤**。
- **`SelectNodes`**：把所有符合條件的標籤都搬出來，回傳一個**清單**（`HtmlNodeCollection`）。

這就像是你跟機器人說：

- **SelectSingleNode**：「去倉庫幫我拿**一個**紅色的箱子。」（它拿完第一個就收工了）
- **SelectNodes**：「把倉庫裡**所有**紅色的箱子都搬出來。」（它會給你一整組清單）

---

## 提取文字 vs. 提取屬性 ⛏️

在 HAP 中，當你定位到一個節點之後，你可以從它身上拿到兩種東西：

### 提取文字：`InnerText`

`InnerText` 會回傳標籤之間夾住的文字內容。

```csharp
// 假設定位到 <p>這是一則重要新聞</p>
var pNode = doc.DocumentNode.SelectSingleNode("//p");
string text = pNode.InnerText; // "這是一則重要新聞"
```

### 提取屬性：`GetAttributeValue`

對於 `<a>` 標籤，我們通常不只想看它的文字，更想知道它連到哪裡（也就是 `href` 屬性）。這時候就要用 `GetAttributeValue`：

```csharp
var linkNode = doc.DocumentNode.SelectSingleNode("//a");

if (linkNode != null)
{
    // 提取標籤之間的文字：例如 "點我前往"
    string text = linkNode.InnerText;

    // 提取標籤內的屬性：例如 "https://google.com"
    string url = linkNode.GetAttributeValue("href", "找不到網址");

    Console.WriteLine($"文字: {text}, 網址: {url}");
}
```

> 💡 **老師的小提醒：預設值的重要性**
> 
> 在 `GetAttributeValue("href", "找不到網址")` 中，第二個參數是「預設值」。如果這個 `<a>` 標籤裡面竟然沒有寫 `href`，程式不會崩潰，而是會回傳你設定的「找不到網址」。這在處理亂糟糟的 HTML 時是非常安全的寫法 🛡️。

### 同理可推：抓取圖片

如果你看到一個圖片標籤 `<img src="cat.jpg" alt="可愛的小貓">`，你可以用同樣的邏輯來定位與提取：

```csharp
// 定位圖片
var imgNode = doc.DocumentNode.SelectSingleNode("//img");

// 提取圖片的檔案路徑
string imgSrc = imgNode?.GetAttributeValue("src", "預設圖片路徑");
```

XPath 寫 `//img` 代表「在文件的任何地方尋找 `<img>` 標籤」，而 `GetAttributeValue("src", ...)` 則是用來拿圖片路徑的方法。

---

## 兩種載入方式：HtmlWeb vs. HtmlDocument 📦

在前面的範例中，我們使用了 `HtmlWeb` 來載入網頁。但你可能也會在其他教學或專案中看到另一種寫法：

```csharp
var doc = new HtmlDocument();
doc.LoadHtml(htmlString);
```

它們的差別在於：你是要 HAP 幫你去**抓網頁**，還是你手上已經有**網頁內容**了。

### 方式一：`HtmlWeb.Load(url)` — 叫外送 🍕

你只要給地址（URL），外送員（`HtmlWeb`）就會負責幫你跑腿，把餐點從遠方的餐廳（伺服器）抓回來，並幫你擺好盤（解析成 `HtmlDocument`）。

```csharp
var web = new HtmlWeb();
var doc = web.Load("https://example.com");
```

- **優點**：方便，一行程式碼搞定連網與解析。
- **缺點**：控制力較低，沒辦法自定義複雜的連線設定（例如特殊的 Header）。

### 方式二：`HtmlDocument.LoadHtml(htmlString)` — 自己下廚 🍳

你手上已經有食材（`htmlString` 字串）了，你只是拿盤子（`HtmlDocument`）把這些食材裝進去並整理好。

```csharp
string html = "<div><h1>哈囉！HAP</h1></div>";

// 1. 準備一個空的盤子
var doc = new HtmlDocument();

// 2. 把食材倒進去
doc.LoadHtml(html);

// 3. 一樣可以用我們學過的 XPath 抓資料
var title = doc.DocumentNode.SelectSingleNode("//h1").InnerText;
Console.WriteLine(title); // 輸出：哈囉！HAP
```

- **優點**：靈活性極高。如果你是用其他方式（例如 `HttpClient`）抓到的網頁原始碼，就必須用這個方法。
- **缺點**：你需要自己先處理好「如何拿到 HTML 字串」這件事。

### 快速對照表

|工具|輸入內容|是否包含網路功能|適用情境|
|---|---|---|---|
|**`HtmlWeb`**|URL (網址)|✅ 是，它會幫你連上網|直接抓取公開網頁|
|**`HtmlDocument`**|String (字串)|❌ 否，它只負責解析|資料已存在記憶體、檔案、或 API 回傳中|

> 💡 **老師的進階小補充**
> 
> 在實務開發中，很多資深工程師更偏好使用 **`HttpClient`** 搭配 **`doc.LoadHtml()`**。這是因為 `HttpClient` 是 .NET 官方最強大的網路工具，可以精細控制 Cookie、代理伺服器 (Proxy) 和超時時間。先把 HTML 抓成字串，再餵給 HAP 解析，這種「分工合作」的方式在處理複雜爬蟲時會更穩定。
> 
> 什麼情況下你可能無法直接用 `HtmlWeb` 抓網址呢？想想看那些需要輸入帳號密碼，或是點擊按鈕後才會出現內容的網頁——這時候就需要先用 `HttpClient` 處理身分驗證，拿到 HTML 字串後再交給 `LoadHtml()` 解析。

---

## 入門篇小結 🎓

恭喜你！到目前為止，你已經學會了 HAP 最核心的知識：

1. **為什麼需要 HAP**：它能處理不規範的 HTML，比 Regex 和 XML 解析器更適合爬蟲場景。
2. **三步驟流程**：載入 → 定位 → 提取。
3. **兩種載入方式**：`HtmlWeb`（幫你抓網頁）和 `HtmlDocument.LoadHtml()`（你自己準備好 HTML 字串）。
4. **兩種提取方式**：`InnerText`（拿文字）和 `GetAttributeValue`（拿屬性）。
5. **Null 檢查的重要性**：永遠要先確認節點存在，再去操作它。

接下來，我們將深入學習 **XPath** 的定位語法——這是讓你在複雜網頁中精準找到目標的關鍵技術。

> 👉 **下一篇**：[[HtmlAgilityPack 教學（二）：XPath 定位語法完全指南]]
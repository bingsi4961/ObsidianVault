---
date: 2026-03-08 19:08
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
#### 📑 [[HtmlAgilityPack 教學（一）：入門篇 — 認識 HAP 與你的第一支爬蟲]]
#### 📑 [[HtmlAgilityPack 教學（二）：XPath 定位語法完全指南]]

---

在前兩篇教學中，你已經學會了 HAP 的核心概念與 XPath 的定位語法。現在，我們要把所有技能串在一起，完成一個完整的實戰任務：**從一個模擬的新聞網頁中，精準抓取熱門新聞，並轉換成 C# 物件，方便後續存入資料庫或顯示在介面上。**

---

## 實戰目標 🎯

我們將使用以下模擬的新聞網頁 HTML。請注意，這個頁面包含了「熱門頭條」和「歷史存檔」兩個區塊——我們的目標是**只抓熱門頭條**，忽略舊聞。

### 練習用的 HTML 原始碼

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

---

## 第一步：定義資料模型 📐

為了讓抓下來的資料好管理，我們先定義一個簡單的「類別 (Class)」來存放一則新聞：

```csharp
public class News
{
    public string Title { get; set; }
    public string Url { get; set; }
    public string ImageUrl { get; set; }
}
```

---

## 第二步：先縮小範圍，再精確打擊 🎯

這是實務上最常用的策略——不要一口氣在整份文件裡面找，而是先定位到你感興趣的「大區塊」，再從裡面逐一提取細節。

### 策略拆解

1. **先定位到「熱門新聞」的大箱子**：用 `//section[@id='hot-news']` 找到目標區塊。
2. **抓出大箱子裡所有的「item」小箱子**：用 `.//div[@class='item']`（注意相對路徑的 `.`）。
3. **對每個小箱子逐一提取**：從中拿出標題、連結、圖片網址。

### 完整程式碼

```csharp
// 建立一個清單來存放抓到的物件
var newsList = new List<News>();

// 1. 先定位到「熱門新聞」的大箱子
var hotNode = doc.DocumentNode.SelectSingleNode("//section[@id='hot-news']");

if (hotNode != null)
{
    // 2. 抓出大箱子裡所有的「item」小箱子
    var items = hotNode.SelectNodes(".//div[@class='item']");

    if (items != null)
    {
        Console.WriteLine($"找到 {items.Count} 則熱門新聞！");

        foreach (var node in items)
        {
            // 3. 從每個小箱子裡提取資料

            // 定位 A 標籤，拿取連結
            var aNode = node.SelectSingleNode(".//a");
            var url = aNode?.GetAttributeValue("href", "未找到 Url");

            // 定位 P 標籤，拿取乾淨的標題文字，並用 Trim() 去掉多餘空白
            var pNode = node.SelectSingleNode(".//p");
            var title = pNode?.InnerText.Trim() ?? "無標題";

            // 定位圖片標籤
            var imgNode = node.SelectSingleNode(".//img");
            var imageUrl = imgNode?.GetAttributeValue("src", "未找到 Image Url");

            var news = new News()
            {
                Title = title,
                Url = url,
                ImageUrl = imageUrl
            };

            newsList.Add(news);
        }
    }
}
```

---

## 常見的踩坑細節 ⚠️

在實作過程中，有幾個新手非常容易犯的小錯誤，這裡一次幫你整理起來：

### 1. 精準選取標題

如果你直接抓 `node.InnerText`，會連同空白、換行甚至圖片的 `alt` 文字都抓進來。建議改抓裡面那個 `<p>` 標籤的文字會更乾淨：

```csharp
// ⚠️ 較髒的寫法：會混入圖片 alt 文字和多餘空白
var title = node.InnerText;

// ✅ 乾淨的寫法：只抓 <p> 標籤的純文字
var title = node.SelectSingleNode(".//p")?.InnerText.Trim() ?? "無標題";
```

### 2. 問號運算子 (`?.`) 的防禦性寫法

在上面的程式碼中，我們使用了 `?.`（Null 條件運算子）。這是因為爬蟲的世界充滿了**不確定性**：

- 萬一某則新聞**剛好沒有圖片**呢？
- `SelectSingleNode` 會回傳 `null`，這時如果直接呼叫方法，程式會報錯崩潰。
- 使用 `?.` 代表：「如果這個節點存在，才去拿屬性；如果不存在，就直接回傳 null」。

這能讓你的爬蟲非常強韌（Robust）！💪

### 3. `SelectNodes` 回傳 null 的陷阱

在使用 `SelectNodes` 時，如果網頁上一個符合條件的標籤都沒有，它會回傳 **`null`**（而不是空清單）。如果你直接對它跑 `foreach` 迴圈，程式就會當掉（報出 `NullReferenceException`）💥。

**安全的寫法一定要先做 null 檢查：**

```csharp
var items = doc.DocumentNode.SelectNodes("//div[@class='item']");

// 先確認真的有抓到東西，再開始跑迴圈
if (items != null)
{
    foreach (var newsNode in items)
    {
        // 處理每一則新聞...
    }
}
```

---

## 用 LINQ 讓程式碼更簡潔 ✨

如果你對 LINQ 有一些基礎，可以把整個「抓取 → 轉換 → 蒐集」的流程串成一條流暢的生產線：

```csharp
var newsList = doc.DocumentNode
    .SelectNodes("//section[@id='hot-news']//div[@class='item']") // 1. 直接定位所有小箱子
    ?.Select(node => new News                                     // 2. 把每個箱子「轉換」成 News 物件
    {
        Title = node.SelectSingleNode(".//p")?.InnerText.Trim() ?? "無標題",
        Url = node.SelectSingleNode(".//a")?.GetAttributeValue("href", "未找到 Url"),
        ImageUrl = node.SelectSingleNode(".//img")?.GetAttributeValue("src", "未找到 Image Url")
    })
    .ToList() ?? new List<News>();                                 // 3. 轉成清單，若沒抓到則給空清單
```

### 為什麼 LINQ 寫法更強大？

|特性|傳統 `foreach`|LINQ 寫法|
|---|---|---|
|**程式碼長度**|較長，需要處理中間變數|極短，一氣呵成|
|**可讀性**|重點分散在各行|專注於「轉換」的邏輯|
|**擴充性**|要過濾資料得再加 `if`|直接串接 `.Where()` 即可|

### 處理 Null 的優雅姿勢

注意到程式碼中用了 `?.Select(...)` 加上 `?? new List<News>()` 嗎？

- **`?.`**：如果前面是 `null`（`SelectNodes` 沒抓到東西），後面就不執行，直接回傳 `null`。
- **`??`**：如果最後結果是 `null`，就給一個空的 `List<News>`。

這樣你的程式永遠不會因為抓不到東西而閃退，這就是「防禦性程式碼」的精髓！🛡️

---

## LINQ 的延伸：過濾與排序 🔍

現在我們手上有一個 `newsList` 了。假設你想**只看標題裡面含有「AI」關鍵字**的新聞，並且**依照標題長度排序**：

```csharp
var filteredNews = newsList
    .Where(news => news.Title.Contains("AI"))
    .OrderBy(news => news.Title.Length)   // 使用 .Length（字串屬性），不是 .Count()
    .ToList();
```

> 💡 **老師的小提醒**：在 C# 中，要取得「字串」的長度，我們使用的是 **`.Length`** 屬性，而不是 `.Count()`。`.Length` 專門給字串和陣列使用，而 `.Count()` 則是 LINQ 提供給 `IEnumerable` 使用的方法。雖然結果相同，但 `.Length` 效能更好且語意更明確。

---

## XPath 一行流：跳過兩段式定位 ⚡

在前面的範例中，我們用了「兩段式」的策略——先抓 `hotNode`，再從裡面用 `.//` 找 `item`。

其實你也可以用**一段式的 XPath** 直接搞定：

```
//section[@id='hot-news']//div[@class='item']
```

這段 XPath 中有兩個 `//`。它的意思拆解如下：

1. `//section[@id='hot-news']`：在整份 HTML 文件的**任何位置**，找到一個 ID 為 `hot-news` 的 `section`。
2. `//div[@class='item']`：在該 `section` **內部的任何地方**（不論是它的兒子、孫子、還是曾孫層級），找到 class 為 `item` 的 `div`。

這種寫法在搭配 LINQ 時特別簡潔：

```csharp
var newsList = doc.DocumentNode
    .SelectNodes("//section[@id='hot-news']//div[@class='item']")
    ?.Select(node => new News
    {
        Title = node.SelectSingleNode(".//p")?.InnerText.Trim() ?? "無標題",
        Url = node.SelectSingleNode(".//a")?.GetAttributeValue("href", ""),
        ImageUrl = node.SelectSingleNode(".//img")?.GetAttributeValue("src", "")
    })
    .ToList() ?? new List<News>();
```

---

## 實戰篇小結 🎓

恭喜你！到目前為止，你已經學會了完成 80% 靜態網頁爬取工作所需的全部技能：

1. **載入網頁**：用 `HtmlWeb.Load` 或 `HtmlDocument.LoadHtml`。
2. **大範圍定位**：用 `SelectNodes` 抓出一整組標籤。
3. **小範圍提取**：用 `SelectSingleNode` 搭配相對路徑 `.//`，從每個節點中精準提取。
4. **拿取寶物**：用 `InnerText`（文字）與 `GetAttributeValue`（屬性）取得資料。
5. **資料整理**：結合 **LINQ** 把亂糟糟的 HTML 變成整齊的 C# 物件。
6. **網址處理**：使用 **`Uri` 類別** 安全地拼接相對路徑，而非 `Path.Combine`。
7. **防禦性程式碼**：善用 `?.`、`??`、`null` 檢查，讓爬蟲不會因為缺少的標籤而崩潰。

---

> 📖 **完整閱讀順序回顧**
> 1. [[HtmlAgilityPack 教學（一）：入門篇 — 認識 HAP 與你的第一支爬蟲]]
> 2. [[HtmlAgilityPack 教學（二）：XPath 定位語法完全指南]]
> 3. HtmlAgilityPack 教學（三）：實戰整合篇 — 從 HTML 到 C_Sharp 物件（本篇）








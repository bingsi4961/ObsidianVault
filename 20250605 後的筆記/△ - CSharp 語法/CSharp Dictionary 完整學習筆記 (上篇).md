---
date: 2026-03-30 17:06
title: C# Dictionary 完整學習筆記 (上篇)
aliases:
tags:
  - CSharp_語法
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[CSharp Dictionary 完整學習筆記 (下篇)|C# Dictionary 完整學習筆記 (下篇)]]

---
## 一、先搞清楚「為什麼需要 Dictionary？」

在正式學語法之前，我想先花幾分鐘跟你講一個很重要的觀念——**「資料結構的選擇，直接影響程式的效能與可讀性」**。

你平常用 `List<T>` 存資料應該已經很習慣了。但我們先來想一個情境：假設你有一個員工列表，主管突然說：「幫我查一下員工編號 1234 叫什麼名字？」

如果資料放在 `List<Employee>` 裡，你的程式碼大概會這樣寫：

```csharp
// 使用 List — 需要逐一掃描，效率低
var employee = employees.FirstOrDefault(e => e.Id == 1234);
```

看起來沒什麼問題，對吧？但是，**這段程式在做的事，其實是從第一筆資料開始，一筆一筆往後找，直到找到目標為止**。如果清單裡有 10,000 筆員工，在最壞的情況下你要比較 10,000 次。

現在換個方式——如果把員工資料放進 `Dictionary`：

```csharp
// 使用 Dictionary — 直接用編號查詢，O(1) 時間
var employeeDict = employees.ToDictionary(e => e.Id);
var employee = employeeDict[1234]; // 直接命中，不需要掃描
```

這個差距有多誇張？我用一個比喻讓你感受一下。

---

### 通訊錄 vs. 便利貼索引：一個讓你秒懂的比喻

想像你有一本有 100 個聯絡人的通訊錄（類比 `List`），以及一張便利貼，上面寫著每個人的名字對應到第幾頁（類比 `Dictionary`）。

**用 `FirstOrDefault`（翻通訊錄）**：

1. 從第一頁開始翻
2. 一頁一頁往後找
3. 直到找到目標，或翻完整本

這就是 **O(n)** 的時間複雜度，n 是資料筆數。100 筆資料，最壞要翻 100 次；10,000 筆要翻 10,000 次。

**用 `Dictionary`（看便利貼索引）**：

1. 先花時間製作索引（建立 Dictionary，只做一次）
2. 之後每次查詢，直接看索引找到目標頁碼
3. 翻到那一頁，完成

這是 **O(1)** 的時間複雜度，不管資料有幾筆，**每次查詢都只需要一步**。

```csharp
// 情境：1,000 筆使用者資料，要重複查詢 1,000 次

// ❌ 用 FirstOrDefault — 總共最多 1,000 × 1,000 = 1,000,000 次比較
foreach (var userId in targetUserIds)
{
    var user = allUsers.FirstOrDefault(u => u.Id == userId); // 每次都掃描全部
}

// ✅ 用 Dictionary — 建立索引 1,000 次 + 查詢 1,000 次 = 2,000 次操作
var userDict = allUsers.ToDictionary(u => u.Id); // 只做一次
foreach (var userId in targetUserIds)
{
    if (userDict.TryGetValue(userId, out var user)) // 每次直接命中
    {
        // ...
    }
}
```

建立 Dictionary 確實需要一些初始成本，但只要後續查詢次數夠多，這個投資就非常值得。特別是在 Portal 系統或 GTS 系統的 API 端點中，經常需要對同一批資料重複查詢時，這個優化效果尤其顯著。

---

## 二、Dictionary 的基本概念

現在你已經知道「為什麼要用 Dictionary」了，讓我們來看它的基本特性。

`Dictionary<TKey, TValue>` 是 C# 中用來儲存**多筆 Key-Value（鍵值）配對資料**的集合型別，類似於其他語言中的 Map 或 Hash Table。

它有四個最重要的特性，請務必記住：

**Key 必須唯一**：每個 Key 在 Dictionary 中只能出現一次。這個限制後面會反覆提到，特別是使用 `ToDictionary` 時最容易踩雷。

**透過 Key 快速查找**：查找時間複雜度是 O(1)，也就是不管資料有多少筆，查詢速度幾乎不變。

**動態調整**：可以在執行時動態新增、修改、刪除資料。

**沒有「第幾筆」的概念**：這點很多新手會搞混。Dictionary 沒有數字索引，你不能用 `dict[0]`、`dict[1]` 這種方式存取，只能透過 Key 存取。

```csharp
// 三種集合型別的對比，讓你一眼看清差異

// List — 用「位置索引」存取（第 0 個、第 1 個…）
List<string> nameList = new List<string> { "張三", "李四", "王五" };
string first = nameList[0]; // ✅ 取得 "張三"

// Array — 同樣用「位置索引」
string[] nameArray = { "張三", "李四", "王五" };
string first = nameArray[0]; // ✅ 取得 "張三"

// Dictionary — 用「Key」存取，沒有位置的觀念
Dictionary<int, string> nameDict = new Dictionary<int, string>
{
    { 1, "張三" },
    { 2, "李四" },
    { 3, "王五" }
};
string name = nameDict[1]; // ✅ 取得 "張三"（Key 是 1，不是「第 0 個」）
// nameDict[0] 會直接爆炸，因為沒有 Key = 0 的資料
```

> ⚠️ **常見誤解提醒**：Dictionary 裡的 Key `1` 跟「第 1 個位置」是不同的概念。`nameDict[1]` 的意思是「找 Key 等於 1 的那筆資料」，而不是「第 1 個元素」。如果你沒有存 Key = 0 的資料，`nameDict[0]` 就會直接丟出 `KeyNotFoundException`。

---

## 三、資料存取方式：四種方法，各有適用場景

Dictionary 有好幾種取值方式，選錯會讓你的程式不是效能差就是意外崩潰。讓我們逐一說明。

### 3-1. 索引器（直接存取）— 確定 Key 存在時才用

```csharp
var students = new Dictionary<int, string>
{
    { 1, "張三" },
    { 2, "李四" },
    { 3, "王五" }
};

// 讀取
string name = students[1];   // 取得 "張三"

// 修改（Key 存在則更新值）
students[1] = "張小三";

// 新增（Key 不存在則新增一筆）
students[4] = "趙七";
```

這是最直覺的寫法，但有個致命缺點：**如果 Key 不存在，會直接拋出 `KeyNotFoundException`，讓程式崩潰**。所以請只在你百分之百確定 Key 一定存在的情況下才用這種寫法。

### 3-2. TryGetValue（安全存取）— 日常開發最推薦

這是我最推薦的寫法，也是效能最好的安全存取方式：

```csharp
// 基本用法
if (students.TryGetValue(1, out string name))
{
    // Key 存在，name 已被賦值
    Console.WriteLine($"學生姓名：{name}");
}
else
{
    // Key 不存在，name 是 null（或預設值）
    Console.WriteLine("找不到該學號的學生");
}
```

你在 Portal 或 GTS 的 Web 開發中，常常會遇到需要有預設值的場景，這樣寫非常實用：

```csharp
// Web API 中常見的寫法：取不到就回傳預設角色
public string GetUserRole(int userId)
{
    if (_userRoles.TryGetValue(userId, out string role))
    {
        return role;
    }
    return "Guest"; // 找不到就給預設角色
}
```

> 💡 **為什麼推薦 `TryGetValue` 而不是先 `ContainsKey` 再取值？** 有些人習慣先 `ContainsKey` 確認存在，再用索引器取值，這樣需要查詢 Dictionary **兩次**。`TryGetValue` 一次就完成「確認存在 + 取值」，效能更好，也更簡潔。

### 3-3. ContainsKey / ContainsValue（只確認是否存在）

```csharp
var config = new Dictionary<string, string>
{
    { "DatabaseConnection", "Server=localhost..." },
    { "ApiKey", "abc123..." }
};

// 確認 Key 是否存在（常用）
if (config.ContainsKey("DatabaseConnection"))
{
    string connString = config["DatabaseConnection"];
    // 這裡用索引器是安全的，因為剛才已確認 Key 存在
}

// 確認 Value 是否存在（較少用，效能是 O(n) 需注意）
if (config.ContainsValue("abc123..."))
{
    Console.WriteLine("找到該 API Key");
}
```

> ⚠️ **`ContainsValue` 的效能陷阱**：`ContainsKey` 是 O(1)，非常快。但 `ContainsValue` 需要逐一掃描所有 Value，是 O(n)，跟 `List` 的 `Contains` 一樣慢。如果你需要頻繁用 Value 反查 Key，考慮維護一個反向 Dictionary。

### 3-4. GetValueOrDefault（.NET Core 的簡潔語法）

這個方法在 .NET Core 3.1（你的 Portal 系統）上可以直接用，但在 .NET Framework 4.8（GTS 系統）上 Dictionary 原生**不支援**，除非自己寫擴充方法。

```csharp
// Portal (.NET Core 3.1) ✅ 原生支援
var settings = new Dictionary<string, string>();

// 找不到 "Theme" 時，直接回傳第二個參數的預設值 "Light"
string theme = settings.GetValueOrDefault("Theme", "Light");

// 只給一個參數：找不到時回傳 null（string 的預設值）
string sidebar = settings.GetValueOrDefault("Sidebar");
```

跟 `TryGetValue` 的差異整理在這裡，讓你一眼看清楚：

|特性|`TryGetValue`|`GetValueOrDefault`|
|---|---|---|
|回傳型別|`bool`（找到/沒找到）|`TValue`（直接是值）|
|主要用途|需要根據「有沒有找到」執行不同邏輯|只需要取值，找不到就用預設值|
|語法繁瑣度|需要 `if` 與 `out` 宣告|一行搞定|
|.NET Framework 4.8（GTS）|✅ 原生支援|❌ 不支援（需自行擴充）|
|.NET Core 3.1（Portal）|✅ 原生支援|✅ 原生支援|

簡單說：**GTS 就用 `TryGetValue`，Portal 兩種都可以，視情況選最清楚的那個**。

---

## 四、遍歷 Dictionary：三種方式

有時候你不是要查詢特定 Key，而是要跑過所有資料。Dictionary 支援幾種遍歷方式：

```csharp
var scores = new Dictionary<string, int>
{
    { "Alice", 85 },
    { "Bob", 92 },
    { "Charlie", 78 }
};

// 方式一：同時取得 Key 和 Value（最常用）
foreach (var item in scores)
{
    Console.WriteLine($"學生：{item.Key}，分數：{item.Value}");
}

// 方式二：C# 7.0+ 解構語法，更簡潔
foreach (var (name, score) in scores)
{
    Console.WriteLine($"學生：{name}，分數：{score}");
}

// 方式三：只遍歷 Keys
foreach (string studentName in scores.Keys)
{
    Console.WriteLine($"學生姓名：{studentName}");
}

// 方式四：只遍歷 Values
foreach (int score in scores.Values)
{
    Console.WriteLine($"分數：{score}");
}
```

> ⚠️ **遍歷時不能修改 Dictionary**：在 `foreach` 迴圈過程中，如果你嘗試新增或刪除 Dictionary 的項目，會拋出 `InvalidOperationException`。如果需要在遍歷過程中修改，先把要刪除/新增的 Key 收集到一個 `List`，迴圈結束後再統一操作。

---

## 五、存取方式選擇指南

最後用一張表幫你整理，面對不同情境該選哪種方式：

| 情境                 | 推薦方法                | 原因          |
| ------------------ | ------------------- | ----------- |
| 確定 Key 一定存在        | `dictionary[key]`   | 直接快速        |
| 不確定 Key 是否存在       | `TryGetValue`       | 安全，避免例外，效能佳 |
| 只需要確認存在性           | `ContainsKey`       | 語意明確        |
| 想要簡潔的預設值語法（Portal） | `GetValueOrDefault` | 一行搞定        |
| 需要遍歷所有資料           | `foreach`           | 支援多種遍歷方式    |
| 效能要求高的大量查找         | `TryGetValue`       | 避免例外處理開銷    |
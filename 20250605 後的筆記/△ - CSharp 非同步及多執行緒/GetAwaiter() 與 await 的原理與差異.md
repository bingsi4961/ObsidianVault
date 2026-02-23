---
date: 2026-02-23 14:55
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

## 前言

在 C# 的非同步程式設計中，我們經常使用 `await` 關鍵字來等待非同步工作完成。但 `await` 其實是一個**語法糖（Syntactic Sugar）**，它背後真正的功臣就是 `GetAwaiter()`。理解這層關係，能幫助我們更深入掌握非同步的運作原理，也能在遇到特殊情境時做出正確的選擇。

---

## 一、什麼是 GetAwaiter()？

在 C# 中，並不是只有 `Task` 才能被 `await`。編譯器的規則是：只要一個類型擁有 `GetAwaiter()` 方法，且該方法回傳的物件符合特定的**模式（Pattern）**，這個類型就是「**可等待的（Awaitable）**」。

回傳的物件稱為 **Awaiter**，它必須具備以下三個成員：

|成員|類型|說明|
|---|---|---|
|`IsCompleted`|Property|告訴編譯器工作是否已經完成|
|`OnCompleted`|Method|當工作完成時，應該執行的回呼（Continuation）|
|`GetResult()`|Method|取得非同步工作的結果；如果發生異常則在此拋出|

---

## 二、await 與 GetAwaiter() 的關係

### 原始程式碼

```csharp
var result = await funcAsync();
Console.WriteLine(result);
```

### 編譯器轉換後的邏輯（簡化版）

```csharp
// 編譯器會將 await 展開成類似以下的邏輯
var awaiter = funcAsync().GetAwaiter();

if (!awaiter.IsCompleted)
{
    // 如果還沒完成，掛起當前方法，註冊回呼函數
    awaiter.OnCompleted(() => 
    {
        var result = awaiter.GetResult();
        Console.WriteLine(result);
    });
    return; // 釋放當前執行緒
}
else
{
    // 如果已經完成，直接拿結果
    var result = awaiter.GetResult();
    Console.WriteLine(result);
}
```

> **重要觀念：** `await` 並不是 `GetAwaiter().GetResult()` 的語法糖，而是**「狀態機（State Machine）」**的語法糖。編譯器會將你的程式碼「切成兩半」，透過 `OnCompleted` 註冊回呼，然後釋放執行緒。

---

## 三、await vs GetAwaiter().GetResult()

這是一個常見的誤解，以下用程式碼釐清兩者的根本差異。

### await 的行為（非同步非阻塞）

```csharp
var awaiter = funcAsync().GetAwaiter();

if (!awaiter.IsCompleted)
{
    // 關鍵：註冊一個回呼（Callback），然後當前 Thread 立即 Return
    awaiter.OnCompleted(() =>
    {
        // 等工作完成後，系統再派一個 Thread 來執行後續程式碼
        var result = awaiter.GetResult();
        ContinueWorkflow(result);
    });
    return; // 這裡的 return 讓 Thread 釋放了，所以不會阻塞
}
```

### GetAwaiter().GetResult() 的行為（同步阻塞）

```csharp
var awaiter = funcAsync().GetAwaiter();

// 跳過了 OnCompleted 的註冊，直接下令取結果
var result = awaiter.GetResult(); // 如果工作還沒完成，這行會卡住當前的 Thread
ContinueWorkflow(result);
```

### 差異整理

|特性|`await`|`GetAwaiter().GetResult()`|
|---|---|---|
|行為|非同步非阻塞：工作未完成時，釋放執行緒|同步阻塞：當前執行緒會卡住，直到工作完成|
|回傳值|直接回傳 `T`|直接回傳 `T`|
|異常處理|拋出原始異常|拋出原始異常（優於 `.Result`）|
|適用場景|絕大多數的非同步開發|無法使用 `async` 的舊版介面或特殊情境|

### 為什麼不用 .Result 或 .Wait()？

如果你必須同步等待一個 `Task`，建議使用 `GetAwaiter().GetResult()` 而不是 `.Result`，原因在於異常處理的差異：

- **`.Result`**：發生錯誤時會將異常封裝在 `AggregateException` 中，追蹤 Error Log 時較麻煩。
- **`GetAwaiter().GetResult()`**：會直接拋出原始異常，讓 Debug 更直覺。

---

## 四、用廚房比喻理解整個機制 🍳

在非同步的世界裡，我們可以用餐廳來比喻各個角色：

|角色|比喻|說明|
|---|---|---|
|**Thread（執行緒）**|服務生|負責外場，接待客人、送餐|
|**Task / Async Function**|廚師|負責內場炒菜（執行非同步工作）|
|**Awaiter**|智慧取餐鈴（Pager）|廚師給服務生的通知裝置|

取餐鈴的三個功能對應 Awaiter 的三個成員：

- **`IsCompleted`（燈號）**：服務生低頭看一下，燈有沒有亮（菜好了嗎？）
- **`OnCompleted`（掛鉤）**：服務生設定「如果燈亮了，請叫任何一位有空的服務生來端菜」
- **`GetResult`（取餐口）**：燈亮後，把餐點從窗口拿走（並確認廚師有沒有把菜燒焦，也就是處理 Exception）

### 用比喻理解 await 與 GetResult() 的差異

|動作|服務生的行為|是否釋放執行緒？|
|---|---|---|
|`await`|拿到取餐鈴，設定好「好了叫我」，然後先去忙別的事|✅ 是（非阻塞）|
|`.GetResult()`|拿到取餐鈴，站在出餐口死盯著看，直到菜出來為止|❌ 否（阻塞）|

### 為什麼這對系統效能很重要？

Thread Pool 的執行緒是共用的。如果你用 `GetResult()` 同步阻塞，當併發量高時（很多人同時點餐），所有服務生都站在廚房門口等，外面就沒人帶位了，系統會變得很慢甚至當機。使用 `await` 的話，就算廚師動作慢，服務生依然可以處理其他人的請求，系統整體吞吐量會高很多。

---

## 五、重點總結

1. **`GetAwaiter()` 是門票**：有了它，物件才能被 `await`。
    
2. **`await` 是自動駕駛**：它幫你處理了「檢查狀態 → 掛起 → 恢復執行」的一連串複雜邏輯，本質上是**狀態機（State Machine）**的語法糖。
    
3. **`await` ≠ `GetAwaiter().GetResult()`**：前者會釋放執行緒（非阻塞），後者會卡住執行緒（阻塞）。千萬不要搞混。
    
4. **優先使用 `await`** 以確保系統的高併發能力。只有在極少數無法將方法簽章改為 `async` 的情況下，才考慮使用 `GetAwaiter().GetResult()`。

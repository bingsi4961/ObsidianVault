---
date: 2025-08-29 15:27
aliases:
tags:
  - CSharp
  - async/await
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
# C# 同步機制筆記: SemaphoreSlim vs. Lock

這份筆記整理了 C# 中兩個核心的同步工具：`SemaphoreSlim` 和 `lock`，涵蓋了它們的概念、用法、區別與最佳實踐。

## 1. `SemaphoreSlim` (咖啡廳座位 ☕)

`SemaphoreSlim` 是一個非同步的同步工具，主要用來**限制能同時存取特定資源的任務數量**。

### 核心概念

- **比喻**：像一間有 N 個座位的咖啡廳。任務（顧客）必須先取得一個座位 (`WaitAsync`) 才能進入，離開時歸還座位 (`Release`)。如果客滿，後來的任務就必須排隊等待。
    
- **用途**：常用於限制 I/O 密集型操作的併發數量，如 API 呼叫、檔案下載、資料庫連線等，防止系統資源超載。
    

### 基本用法範例

最安全、標準的寫法是搭配 `try...finally` 區塊，確保資源一定會被釋放。

```csharp
// 建立一個最多只允許 3 個任務同時進入的號誌
private static SemaphoreSlim semaphore = new SemaphoreSlim(3);

public async Task AccessResourceAsync()
{
    // 等待「綠燈」（要求一個位置），如果客滿會在此非同步等待
    await semaphore.WaitAsync();
    try
    {
        // --- 這裡的程式碼區塊 ---
        // --- 同一時間最多只會有 3 個任務能執行到這裡 ---
        Console.WriteLine($"任務 {Task.CurrentId} 進入了控制區。");
        await Task.Delay(1000); // 模擬工作
    }
    finally
    {
        // 釋放「綠燈」（歸還位置），讓下一個任務可以進來
        // 這是至關重要的一步，可防止資源洩漏導致的死鎖
        semaphore.Release();
        Console.WriteLine($"任務 {Task.CurrentId} 離開了。");
    }
}
```

> **重點**: `Release()` 必須放在 `finally` 區塊，確保無論 `try` 區塊是否發生例外，鎖都會被釋放。

### `Wait()` vs. `WaitAsync()`

- `Wait()`: **同步**等待，會**阻塞**當前執行緒，直到取得鎖。
    
- `WaitAsync()`: **非同步**等待，**不會阻塞**執行緒。執行緒會被釋放回執行緒池處理其他工作，是現代非同步程式的首選。
    

### 監控可用名額 `CurrentCount`

`SemaphoreSlim` 提供一個非常有用的屬性：`CurrentCount`。

- **功能**：它會回傳**目前還剩下多少個可用的名額**。
    
- **用途**：主要用於監控和除錯，可以即時了解號誌的負載狀況。它的值會在 `WaitAsync` 成功後減 1，在 `Release` 後加 1。
    

### 實用範例：限制併發下載數量 (含 `CurrentCount`)

```csharp
// 限制最多只能同時執行 5 個下載任務
private static SemaphoreSlim downloaderSemaphore = new SemaphoreSlim(5);

// 模擬下載單一檔案的方法
public async Task DownloadFileAsync(int fileId)
{
    Console.WriteLine($"[檔案 {fileId,3}] - 排隊等待下載... (目前可用名額: {downloaderSemaphore.CurrentCount})");
    await downloaderSemaphore.WaitAsync();
    try
    {
        // 成功進入後，可以透過 (初始數量 - CurrentCount) 來觀察目前的併發數
        Console.WriteLine($"✅ [檔案 {fileId,3}] - 開始下載... (目前併發數: {5 - downloaderSemaphore.CurrentCount})");
        await Task.Delay(2000); // 模擬下載時間
        Console.WriteLine($"✔️ [檔案 {fileId,3}] - 下載完成！");
    }
    finally
    {
        downloaderSemaphore.Release();
    }
}

// 主程式：一次啟動 100 個任務，但 SemaphoreSlim 會確保併發數不超過 5
public async Task DownloadAllFilesAsync()
{
    var allTasks = new List<Task>();
    for (int i = 1; i <= 100; i++)
    {
        allTasks.Add(DownloadFileAsync(i));
    }
    await Task.WhenAll(allTasks);
}
```

## 2. `lock` (廁所鑰匙 🔑)

`lock` 是 C# 的一個關鍵字，用於確保一段程式碼區塊在同一時間**只會被一個執行緒執行**。

### 核心概念

- **比喻**：像一把廁所的鑰匙。一次只能有一個人（執行緒）拿到鑰匙進去。其他人必須在門口排隊（同步阻塞）。
    
- **用途**：保護共享的**記憶體中物件**，防止因多執行緒同時寫入而導致的資料錯亂。例如保護 `List`、`Dictionary` 或其他共享變數。
    

### 基本用法範例

`lock` 關鍵字的語法非常簡潔，它會自動處理 `try...finally` 和鎖的釋放。

```csharp
// 最佳實踐：建立一個私有的、唯讀的物件當作「鑰匙」
private readonly object _lockObject = new object();
private int _sharedCounter = 0;

public void IncrementCounter()
{
    // 鎖上這個「鑰匙」
    lock (_lockObject)
    {
        // 在這個區塊內，程式碼是執行緒安全的
        _sharedCounter++;
    } // 離開區塊時，鎖會自動釋放
}
```

### `lock` 的最佳實踐：避免死鎖

> **黃金法則**: 永遠鎖定一個 `private readonly object`，這個物件的唯一用途就是當作鎖。

- **不要 `lock (this)` 或 `lock (公開的物件)`**：因為外部程式碼也可能鎖定同一個公開的物件，如果你們鎖定資源的順序不同，就極有可能導致**死鎖 (Deadlock)**。
    
- **死鎖 (Deadlock)**：指兩個或多個執行緒互相持有對方需要的資源，並等待對方釋放，導致所有執行緒都無限期地卡住。
    

#### 死鎖程式碼範例

以下範例展示了兩個執行緒如何因為鎖定順序相反，而導致互相等待，形成死鎖。

```csharp
private static readonly object lockA = new object();
private static readonly object lockB = new object();

public static void Thread1_Work()
{
    lock (lockA) // 執行緒 1: 先拿 A
    {
        Thread.Sleep(100);
        lock (lockB) // 再拿 B
        {
            // ...
        }
    }
}

public static void Thread2_Work()
{
    lock (lockB) // 執行緒 2: 先拿 B
    {
        Thread.Sleep(100);
        lock (lockA) // 再拿 A
        {
            // ...
        }
    }
}
// 同時執行 Thread1_Work 和 Thread2_Work 將導致死鎖。
```

## 3. `SemaphoreSlim` vs. `Lock` 總結

|   |   |   |
|---|---|---|
|**特性**|**lock (Monitor)**|**SemaphoreSlim**|
|**允許數量**|**永遠只有 1 個**執行緒|**可以自訂 N 個** (N ≥ 1) 任務|
|**等待方式**|**同步阻塞** (執行緒會卡住)|**支援非同步等待** (`WaitAsync`)|
|**`async/await`**|**不支援** (在 `lock` 內 `await` 會報錯)|**完美支援** (它的主要使用情境)|
|**常見用途**|保護**記憶體**中的物件 (List, Dictionary)|限制對**外部資源**的存取 (API, DB)|

### 經驗法則

- 如果你的程式碼區塊是**同步**的 (沒有 `await`)，請優先使用 `lock`。
    
- 如果你的程式碼區塊是**非同步**的 (`async/await`)，請使用 `SemaphoreSlim`。
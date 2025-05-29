---
title: TPL（Task Parallel Library）/ Task.Run
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5]

---

# TPL（Task Parallel Library）簡介

TPL是C#/.NET框架中的任務並行庫（Task Parallel Library），它在.NET Framework 4.0中首次引入，用於簡化並行和非同步程式設計。TPL讓開發者能夠更容易地撰寫多執行緒和非同步程式，而不必直接處理執行緒的建立與管理的複雜細節。

## TPL的核心概念

TPL的核心是`Task`類別，它代表一個非同步操作。與傳統的`Thread`類別相比，`Task`提供了更高層級的抽象，並能更有效地利用系統資源。


## 基本用法範例

### 建立和啟動任務

```csharp
// 建立一個簡單的任務
Task task = new Task(() => {
    Console.WriteLine("任務正在執行...");
    // 執行一些工作
});

task.Start();    // 啟動任務
task.Wait();    // 等待任務完成
```

```csharp
static void Main()
{
    var task = new Task(MyTask);
    task.Start();

    for (int i = 0; i < 500; i++)
    {
        Console.Write(".");
    }
}

static void MyTask()
{
    for (int i = 0; i < 500; i++)
    {
        Console.Write("[" + Thread.CurrentThread.ManagedThreadId + "]");
    }
}
```

```csharp
// 或更簡潔的方式：使用Task.Run
Task taskRun = Task.Run(() => {
    Console.WriteLine("使用Task.Run啟動的任務");
});

// ★ 如果跟 UI有關的非同步，Wait會造成 DeadLock
taskRun.Wait();    // 等待taskRun完成
```

### 使用Task<TResult>返回結果

```csharp
// 建立一個返回整數結果的任務
Task<int> calculationTask = Task.Run(() => {
    // 假設這是一個耗時的計算
    int result = 0;
    for (int i = 1; i <= 100; i++) {
        result += i;
    }
    return result;
});

// 取得結果（如果任務尚未完成，這裡會等待）
// ★ 如果跟 UI有關的非同步，Result會造成 DeadLock
int sum = calculationTask.Result;
Console.WriteLine($"計算結果: {sum}");
```

## 使用 TAP (async/await) 與 TPL

TPL與C#的`async`和`await`關鍵字緊密結合，這使得非同步程式設計變得更加簡單：

```csharp
// 非同步方法範例
public async Task<string> DownloadDataAsync(string url) {
    // HttpClient是專為非同步操作設計的
    using (HttpClient client = new HttpClient()) {
        // 非同步下載資料，不會阻塞UI執行緒
        string result = await client.GetStringAsync(url);
        return result;
    }
}

// 呼叫非同步方法
public async Task ProcessDataAsync() {
    Console.WriteLine("開始下載資料...");
    string data = await DownloadDataAsync("https://example.com/api/data");
    Console.WriteLine($"下載完成，資料長度: {data.Length}");
    // 處理資料...
}
```

## TPL的進階功能

TPL還提供了許多進階功能，如：

1. **任務延續**：使用`ContinueWith`方法在一個任務完成後執行另一個任務
2. **任務取消**：通過`CancellationToken`支援取消長時間執行的操作
3. **任務例外處理**：處理非同步操作中發生的例外
4. **任務協調**：使用`Task.WhenAll`和`Task.WhenAny`協調多個非同步操作

## 為什麼要使用TPL？

TPL相較於傳統執行緒管理提供了許多優勢：

1. **更高效的資源使用**：TPL使用執行緒集區來管理執行緒，避免過多執行緒的建立和銷毀
2. **更簡潔的程式碼**：減少了手動執行緒同步的複雜性
3. **更好的可擴展性**：TPL會根據系統資源動態調整並行度
4. **與非同步模式的整合**：與C#的async/await完美配合

TPL是現代C#程式設計中處理並行和非同步操作的基礎技術，掌握它對於編寫高效能、回應靈敏的應用程式至關重要。
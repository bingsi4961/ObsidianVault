---
title: async/await vs ContineWith
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5]

---

```csharp
static async Task Main(string[] args)
{
    // async / await 版本
    // T + 0ms: Main 調用 DownloadPageStringAsync_1
    // T + 1ms: 創建 HttpClient 並發起網路請求
    // T + 2ms: 遇到 await，暫停方法執行，返回 Task 給 Main
    // T + 3ms: Main 方法等待 task1 完成
    // T + 500ms: GetStringAsync() 網路請求完成
    // T + 501ms: DownloadPageStringAsync_1 方法恢復執行
    // T + 502ms: 返回結果，完成返回給 Main 的 Task
    // T + 503ms: Main 方法獲得結果並繼續執行

    // ContinueWith 版本
    // T + 0ms: Main 調用 DownloadPageStringAsync_2
    // T + 1ms: 創建 HttpClient、TaskCompletionSource，發起網路請求
    // T + 2ms: 註冊 ContinueWith 回調到請求任務上
    // T + 3ms: 返回 tcs.Task 給 Main
    // T + 4ms: Main 在 tcs.Task 上註冊另一個 ContinueWith
    // T + 5ms: Main 方法等待其 ContinueWith 完成
    // T + 500ms: GetStringAsync() 網路請求完成
    // T + 501ms: 執行 DownloadPageStringAsync_2 中註冊的回調
    // T + 502ms: 回調處理結果並完成 tcs.Task
    // T + 503ms: 執行 Main 中註冊的回調並輸出結果
    // T + 504ms: Main 方法恢復執行


    string url = "https://www.huanlintalk.com/2024/11/csharp-dotnet-to-async-or-not.html";

    Task<string> _task1 = DownloadPageStringAsync_1(url);
    string content = await _task1;
    Console.WriteLine(" async/await 網頁內容總共為 {0} 個字元。", content.Length);

    // ContinueWith 使用回呼（callback）的方式定義「當前一個任務完成後要做什麼」。
    // 在 tcs.Task 上註冊了另一個 ContinueWith 回調
    // 這個回調會等待 tcs.Task 完成後執行
    // Main 方法的控制流遇到 await，暫停執行等待這個 ContinueWith 完成
    await DownloadPageStringAsync_2(url)
        .ContinueWith(task =>
        {
            Console.WriteLine(" ContinueWith 網頁內容總共為 {0} 個字元。", task.Result.Length);
        });
}

static async Task<string> DownloadPageStringAsync_1(string url)
{
    using (HttpClient httpClient = new HttpClient())
    {
        Task<string> _task2 = httpClient.GetStringAsync(url);
        string content = await _task2;
        return content;
    }
}

static Task<string> DownloadPageStringAsync_2(string url)
{
    var httpClient = new HttpClient();
    var tcs = new TaskCompletionSource<string>();

    // ContinueWith 則更直接地表達了底層的 Task 操作模型：
    // 一個 Task 完成後，立即在 TaskScheduler 排程中執行指定的回呼函數
    // 回呼函數是作為新的工作項目被排程的，不會自動捕捉同步處理的上下文
    // 開發者需要手動處理任務的狀態（完成、失敗、取消）和資源釋放

    // 調用 httpClient.GetStringAsync(url) 獲得一個尚未完成的 Task
    // 立即在該 Task 上註冊一個 ContinueWith 回調
    // 方法返回 tcs.Task（目前尚未完成的任務）給 Main
    // 控制權立即返回到 Main 方法，不等待網路請求完成

    httpClient.GetStringAsync(url)
        .ContinueWith(task =>
        {
            // 尚未執行，只是被註冊到 Task 中
            // 將在網路請求 Task 完成後被調度執行
            try
            {
                if (task.IsCompleted) tcs.SetResult(task.Result);
                if (task.IsFaulted) tcs.SetException(task.Exception);
                if (task.IsCanceled) tcs.SetCanceled();
            }
            finally
            {
                httpClient.Dispose();
            }
        });

    return tcs.Task;
}
```
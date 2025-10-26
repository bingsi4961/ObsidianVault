---
date: 2025-10-26 22:29
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

```csharp
// 1. 不能用 async Task，改回 void
private void myButton_Click(object sender, EventArgs e)
{
    // 2. 手動捕獲當前的 UI 上下文 (SynchronizationContext)
    var uiContext = SynchronizationContext.Current;
    
    // 3. 呼叫函式庫方法，這會傳回一個 Task 物件
    Task<string> dataTask = MyAwesomeLibrary.GetDataAsync();
    
    // 4. 「await」的真正替代品：.ContinueWith
    //    意思是「當 dataTask 完成後，請執行以下...」
    dataTask.ContinueWith(task => 
    {
        // --- 這段程式碼可能在「任何執行緒」上執行 ---
        
        // 5. 從完成的 Task 中取得結果
        string result = task.Result; 
        
        // 6. 關鍵步驟：
        //    我們不能在這裡直接更新 myLabel.Text (會崩潰！)
        //    必須使用一開始捕獲的 uiContext
        //    把更新任務「Post」回 UI 執行緒
        
        // ✅ 標準寫法：透過 state 參數傳遞資料
        uiContext.Post(state => 
        {
            // --- 這段程式碼現在保證在 UI 執行緒上安全執行 ---
            // 'state' 參數就是我們傳入的 'result'
            // (需要做一次型別轉換)
            string data = (string)state;
            myLabel.Text = data;
            
        }, result); // <-- 把 result 當作 state 參數傳遞
        
        // ⚠️ 也可以依賴閉包 (Closure)，但不建議
        // uiContext.Post(_ => 
        // {
        //     myLabel.Text = result; // 捕獲外層變數
        // }, null);
        
    });
}

public static class MyAwesomeLibrary
{
    // 1. 方法不再是 async
    //    它只是一個「傳回 Task 的一般方法」
    public static Task<string> GetDataAsync()
    {
        // 2. Task.Run 會立刻把工作排入 ThreadPool
        //    並「立刻」傳回一個 Task 物件
        return Task.Run(() => 
        {
            // --- 這整段程式碼都在背景執行緒 (Thread Pool) 上執行 ---
            
            // 3. 以前沒有 Task.Delay，我們會用 Thread.Sleep 
            //    (在背景執行緒 Sleep，不會卡住 UI)
            Thread.Sleep(1000); 
            
            // 4. 當 return 時，Task.Run 會自動把這個 "Task<string>"
            //    標記為「已完成」，並把結果包進去
            return "這是從伺服器拿到的資料";
        });
    }
}
```

這個範例，用來說明在沒有 `async/await` 的情況下，如何手動處理執行緒上下文 (Context) 的切換。

這段程式碼主要涉及兩種執行緒：
1. **UI 執行緒 (UI Thread)**：唯一的，負責處理所有使用者介面操作（點擊、繪製、更新文字）的執行緒。    
2. **背景執行緒 (Thread Pool Thread)**：由 .NET 執行階段管理的一組執行緒，用於執行耗時的背景工作，以免卡住 UI。    

---
### `myButton_Click` 方法 (事件處理常式)

```csharp
private void myButton_Click(object sender, EventArgs e)
{
```

- **執行緒：** `UI 執行緒`    
- **說明：** 所有的 UI 事件（例如按鈕點擊）必定是從 UI 執行緒開始執行的。
    
```csharp
    var uiContext = SynchronizationContext.Current;
```

- **執行緒：** `UI 執行緒`    
- **說明：** 承上，程式仍在 UI 執行緒上，此行程式碼會「捕獲」當前的上下文，也就是 UI 執行緒的上下文。
    
```csharp
    Task<string> dataTask = MyAwesomeLibrary.GetDataAsync();
```

- **執行緒：** `UI 執行緒`    
- **說明：**    
    1. 程式在 UI 執行緒上呼叫 `GetDataAsync()`。        
    2. `GetDataAsync()` 內部的 `Task.Run` 會「立即」將工作丟入背景執行緒池 (Thread Pool)。        
    3. `GetDataAsync()` 「立即」傳回一個 `Task<string>` 物件（一個「承諾」），<mark style="background: #FFF3A3A6;">此時它還在 UI 執行緒上</mark>。        
    4. UI 執行緒繼續往下執行，完全沒有被 `Task.Run` 內部的工作（`Thread.Sleep`）卡住。        

```csharp
    dataTask.ContinueWith(task => 
    {
```

- **執行緒：** `UI 執行緒`    
- **說明：** 這一行程式碼的「註冊」動作本身是在 UI 執行緒上完成的。它告訴 Task Scheduler：「嘿，等 `dataTask` 完成後，請幫我執行這個 Lambda 裡面的程式碼。」    
- **重點：** `ContinueWith` 內部的程式碼 `task => { ... }` **尚未執行**。UI 執行緒註冊完畢後，`myButton_Click` 方法就執行完畢了，UI 執行緒恢復自由，可以繼續處理其他 UI 事件。    

---
### `MyAwesomeLibrary.GetDataAsync` 方法 (函式庫)

在 `myButton_Click` 呼叫它時：

```csharp
public static Task<string> GetDataAsync()
{
    return Task.Run(() => 
    {
```

- **執行緒 (呼叫時)：** `UI 執行緒`    
- **說明：** 如前所述，`Task.Run` 本身是在 UI 執行緒上被呼叫的。它會「立即」傳回一個 `Task` 物件給 `myButton_Click`。    
- **執行緒 (Lambda 內部)：** `背景執行緒 (Thread Pool 執行緒)`    
- **說明：** `Task.Run` 的定義就是將 `() => { ... }` 裡面的所有程式碼，都丟到 Thread Pool 中去執行。    

```csharp
        Thread.Sleep(1000); 
```

- **執行緒：** `背景執行緒 (Thread Pool 執行緒)`    
- **說明：** 某一個背景執行緒被「阻塞」了 1 秒鐘。因為這不是 UI 執行緒，所以使用者的介面依然是流暢的。    

```csharp
        return "這是從伺服器拿到的資料";
    });
}
```

- **執行緒：** `背景執行緒 (Thread Pool 執行緒)`    
- **說明：**    
    1. 背景執行緒執行完 `Sleep`。        
    2. 背景執行緒準備好要回傳的字串。        
    3. 當 `return` 執行時，這個背景執行緒會通知 `Task` 物件：「我完成了，結果是這個字串」。        
    4. `Task` 物件的狀態變為「已完成」(Completed)。        
    5. 這個「完成」事件會觸發先前在 `myButton_Click` 中註冊的 `ContinueWith`。        

---
### `dataTask.ContinueWith` 的回呼 (Callback)

在 `GetDataAsync` 完成後，`ContinueWith` 內的程式碼會被觸發：

```csharp
    dataTask.ContinueWith(task => 
    {
        // --- 這段程式碼可能在「任何執行緒」上執行 ---
```

- **執行緒：** <mark style="background: #FFF3A3A6;">背景執行緒 (Thread Pool 執行緒)</mark>    
- **說明：**    
    - 由於 `dataTask` (來自 `Task.Run`) 是在背景執行緒上完成的，預設的 `ContinueWith` 會直接使用同一個背景執行緒（或另一個可用的背景執行緒）來接續執行，這樣最有效率。        
    - 這就是您註解「可能在任何執行緒上執行」的意思——它**幾乎可以肯定不在 UI 執行緒上**。        

```csharp
        string result = task.Result; 
```

- **執行緒：** `背景執行緒 (Thread Pool 執行緒)`    
- **說明：** 承上，程式仍在背景執行緒。`task.Result` 是安全的，因為 `ContinueWith` 保證 `task` 已經完成了，所以讀取 `.Result` 不會造成鎖死。    

```csharp
        uiContext.Post(state => 
        {
```

- **執行緒：** `背景執行緒 (Thread Pool 執行緒)`    
- **說明：** 程式在背景執行緒上，呼叫了 `uiContext.Post` 方法。    
- **重點：** `Post` 方法本身是一個「非同步」操作。它會把 `state => { ... }` 這個委派（delegate）打包成一個訊息，然後「發佈」到 `uiContext`（也就是 UI 執行緒）的訊息佇列 (Message Queue) 中，然後**立即返回**，讓背景執行緒繼續往下執行 (雖然這個範例中 `ContinueWith` 已經沒事做了)。    

```csharp
        }, result);
    });
} // myButton_Click 結束
```

---
### `uiContext.Post` 的回呼 (Callback)

UI 執行緒在處理完它手邊的工作（例如繪製、其他事件）後，會從它的訊息佇列中取出由 `Post` 發送的那個訊息：

```csharp
        uiContext.Post(state => 
        {
            // --- 這段程式碼現在保證在 UI 執行緒上安全執行 ---
            string data = (string)state;
            myLabel.Text = data;
        }, result); 
```

- **執行緒：** `UI 執行緒`    
- **說明：** `Post` 內部的所有程式碼 `state => { ... }`，現在保證回到了 UI 執行緒上執行。    
- `string data = (string)state;`：在 `UI 執行緒` 上執行型別轉換。    
- `myLabel.Text = data;`：在 `UI 執行緒` 上安全地更新 UI 控制項。    

### 總結 (執行緒切換流程)

1. **UI 執行緒**：啟動 `myButton_Click` -> 捕獲 `uiContext` -> 呼叫 `GetDataAsync`。
    
2. **UI 執行緒**：`GetDataAsync` 呼叫 `Task.Run` 並「立即」傳回 `Task`。
    
3. **背景執行緒**：`Task.Run` 內部的 `Thread.Sleep` 開始執行。
    
4. UI 執行緒：(幾乎在同時) ContinueWith 註冊完成 -> myButton_Click 結束 -> UI 執行緒空閒。
    
    ... (1 秒後) ...
    
5. **背景執行緒**：`Thread.Sleep` 結束 -> `return "..."` -> `dataTask` 標記為完成。
    
6. **背景執行緒**：(因為 Task 完成了) `ContinueWith` 的 Lambda 被觸發 -> 讀取 `task.Result` -> 呼叫 `uiContext.Post`。
    
7. **UI 執行緒**：(在它方便的時候) 從訊息佇列中取出 `Post` 的任務 -> 執行 Lambda -> 更新 `myLabel.Text`。
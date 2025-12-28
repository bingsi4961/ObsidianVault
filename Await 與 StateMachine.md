---
date: 2025-12-28 15:01
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

## 1. 為什麼需要非同步程式設計

### 1.1 同步 vs 非同步的本質差異

#### 生活化比喻

想像你在一家餐廳當廚師：

**同步執行（Synchronous）**

- 你點火煮麵，然後站在爐火旁發呆 5 分鐘
- 這期間你不能點餐、不能切菜，直到麵煮好
- 這就是「阻塞（Blocking）」，效率極低

**非同步執行（Asynchronous）**

- 你點火煮麵，設個鬧鐘
- 轉身去幫下一位客人點餐
- 等鬧鐘響了，你再回來處理麵
- 這就是 `await` 的精神：「不等待，先去做別的事，等好了再回來」

#### 技術層面比較

|特性|同步執行|非同步執行 (await)|
|---|---|---|
|**執行緒行為**|死守現場，直到工作完成|暫放工作，先去處理別的事|
|**資源利用**|耗費執行緒等待，浪費資源|釋放執行緒，提高吞吐量|
|**使用者體驗**|介面可能卡死 (Not Responding)|介面保持流暢，可同時操作|
|**錯誤處理**|直接拋出錯誤|拆開 `AggregateException` 呈現原貌|

### 1.2 核心角色定義

在理解非同步程式設計時，我們可以用餐廳的比喻來對應：

- **執行緒（Thread）**：就是服務生 🧑‍🍳
- **耗時任務（Task）**：像是廚房製作一份需要烤 30 分鐘的排餐
- **await**：服務生把點單交給廚房後，轉身去服務其他客人的動作
- **執行緒池（Thread Pool）**：餐廳的服務生團隊
- **狀態機（State Machine）**：服務生的筆記本，記錄每桌的進度

---

## 2. 核心觀念與基礎知識

### 2.1 async 與 await 的基本語法

```csharp
public async Task OrderCoffeeAsync()
{
    Console.WriteLine("1. 開始煮咖啡...");
    
    // 遇到 await：檢查狀態、紀錄位置、釋放執行緒
    // 檢查 Task 是否已完成
    // 紀錄已執行到的程式碼位置
    // 先邏輯釋放(Caller)，再實體釋放(Thread Pool)
    await Task.Delay(3000); // 模擬煮咖啡需要 3 秒
    
    // 3 秒後：回到狀態機紀錄的位置繼續執行
    Console.WriteLine("2. 咖啡煮好了！");
}
```

### 2.2 Task 的本質

`Task` 代表一個可能還沒完成的操作，它包含：

- 操作的狀態（未開始、執行中、已完成、已取消、發生錯誤）
- 操作的結果（如果有的話）
- 錯誤資訊（如果發生錯誤）

### 2.3 執行緒與 Task 的關係

**重要觀念**：Task 不等於 Thread

- 一個 Task 可能由多個 Thread 接力完成
- 多個 Task 可能由同一個 Thread 處理
- Task 是邏輯上的工作單位，Thread 是實際執行的工人

---

## 3. await 的完整運作機制

### 3.1 await 執行時的五個階段

#### 第一階段：啟動與檢查

```csharp
// 當程式執行到這行
await SomeAsyncMethod();
```

**發生的事情**：

1. 首先呼叫 `SomeAsyncMethod()`
2. 該方法執行直到遇到它內部的第一個 `await`
3. 檢查回傳的 `Task.IsCompleted`
    - 如果 `true`：直接同步繼續執行（優化路徑）
    - 如果 `false`：進入暫停流程

**關鍵觀察**：不是看到 `await` 就立刻暫停，而是要等方法回傳 Task 且該 Task 未完成

#### 第二階段：暫停與紀錄（進入狀態機）

當任務尚未完成時：

**建立「書籤」**：

```csharp
// 編譯器產生的虛擬程式碼
class GeneratedStateMachine
{
    int state;           // 目前執行到哪個 await（書籤頁碼）
    TaskAwaiter awaiter; // 正在等待的任務
    
    // 區域變數
    string localVar1;
    int localVar2;
    
    // 原始方法邏輯
    void MoveNext() { ... }
}
```

**紀錄內容**：

- 目前執行到哪一行（state）
- 所有區域變數的值
- 正在等待的 Task 參考

#### 第三階段：釋放執行緒

這是非同步最神奇的地方，包含兩個層次：

**1. 邏輯上的釋放**

```csharp
// Thread A 正在執行
await Task.Delay(1000); // <- Thread A 在這裡「邏輯暫停」
// Thread A 將控制權交回給呼叫者
```

- 方法暫停執行
- 將進度保存在狀態機 (state)
- 回傳 Task 給呼叫者 (🚨await 所在的 Task (尚未完成))
- UI 可以繼續回應

**2. 實體上的釋放**

```csharp
// 當最上層的方法也 await 後
// Thread A 完全沒事做了
// Thread A 回到執行緒池
```

- 執行緒完成所有存檔動作
- 回到執行緒池待命
- 可以去處理其他請求

#### 第四階段：任務完成與回歸

```csharp
// 1 秒後，計時器完成
// 計時器透過 callback 方式通知狀態機：「時間到了，可以繼續了」
// 狀態機請 Thread Pool：「給我一個執行緒來繼續工作」
// Thread Pool 派出 Thread B
// Thread B 讀取狀態機記錄的「書籤」（在哪個 await 暫停）
// Thread B 恢復所有變數，從 await 下一行繼續執行
```

**執行緒的選擇**：

- 可能是原本的 Thread A（如果它剛好有空）
- 可能是任何其他執行緒
- 由執行緒池自動調度

#### 第五階段：錯誤處理（拆箱作業）

```csharp
try
{
    await SomeMethodThatThrows();
}
catch (FileNotFoundException ex) // 直接捕捉原始異常
{
    // await 已經幫你從 AggregateException 拆出真正的錯誤
}
```

**拆箱過程**：

1. 非同步任務的錯誤會被包在 `AggregateException`
2. `await` 自動解包，取出第一個（通常也是唯一的）內部異常
3. 讓你用傳統 try-catch 處理

### 3.2 await 的額外工作

#### 捕捉上下文（Capture Context）

```csharp
// 在 UI 執行緒上
await DownloadDataAsync();
// await 會試圖回到 UI 執行緒
UpdateUI(); // 這樣才能安全更新介面
```

**SynchronizationContext 的角色**：

- 記住原本的執行環境（UI 執行緒、ASP.NET 請求等）
- 任務完成後盡量回到相同環境
- 可用 `ConfigureAwait(false)` 關閉此行為

---

## 4. 執行緒的生命週期詳解

### 4.1 完整範例程式碼

```csharp
static async Task Main()
{
    Console.WriteLine("Line-1");
    await ProcessData();
    Console.WriteLine("Line-2");
}

static async Task ProcessData()
{
    Console.WriteLine("Line-3");
    await DoHeavyWorkAsync();
    Console.WriteLine("Line-4");
}

static async Task DoHeavyWorkAsync()
{
    Console.WriteLine("Line-5");
    await Task.Delay(1000);
    Console.WriteLine("Line-6");
}
```

### 4.2 執行流程詳細分解

#### 階段一：同步開始（一路向下）

|步驟|程式碼|執行緒|說明|
|---|---|---|---|
|1|`Console.WriteLine("Line-1")`|Thread A|Main 開始執行|
|2|`await ProcessData()`|Thread A|**進入** ProcessData（還沒暫停）|
|3|`Console.WriteLine("Line-3")`|Thread A|繼續同步執行|
|4|`await DoHeavyWorkAsync()`|Thread A|**進入** DoHeavyWorkAsync|
|5|`Console.WriteLine("Line-5")`|Thread A|仍然同步執行|

**關鍵觀念**：即使經過多個 async 方法，<mark style="background: #FFF3A3A6;">在碰到第一個「真正需要等待」的操作前，都是同一個執行緒在執行</mark>

#### 階段二：暫停與連鎖釋放

```
執行流程圖：
Thread A 執行路徑：Main → ProcessData → DoHeavyWorkAsync
                                              ↓
                                    await Task.Delay(1000) ← 撞牆了！
                                              ↓
                          DoHeavyWorkAsync 回傳 Task（未完成）/ 不是 await 回傳 Task（未完成）
                                              ↓
                            ProcessData 收到未完成的 Task
                                              ↓
                              ProcessData 也回傳 Task
                                              ↓
                                Main 收到未完成的 Task
                                              ↓
                              Thread A 回到執行緒池
```

#### 階段三：恢復執行（連貫性）

```
1000ms 後...
計時器：「時間到了！」
執行緒池：「Thread B，你去處理」

Thread B 的執行路徑：
1. 讀取 DoHeavyWorkAsync 的狀態機 → 執行 Line-6
2. DoHeavyWorkAsync 完成 → 觸發 ProcessData 的 await
3. 繼續執行 Line-4（Thread B 沒有回池子）
4. ProcessData 完成 → 觸發 Main 的 await
5. 繼續執行 Line-2（Thread B 一路做到底）
```

**重要觀察**：<mark style="background: #FFF3A3A6;">Thread B 會盡可能把所有後續工作一次完成，不會每個 await 都換執行緒</mark>

### 4.3 執行緒釋放的詳細時機

|程式碼階段|執行者|狀態變更|詳細說明|
|---|---|---|---|
|開始執行|Thread A|**佔用中**|執行 Line-1, 3, 5|
|`await Task.Delay`|Thread A|**釋放中**|遇到未完成任務，開始存檔|
|連鎖釋放|Thread A|**釋放中**|沿呼叫堆疊回傳 Task|
|回到池子|Thread A|**已釋放**|Thread A 可處理其他請求|
|等待期間|(無)|**暫停**|只有計時器在跑|
|任務完成|Thread B|**重新指派**|從池子派出有空的執行緒|
|繼續執行|Thread B|**佔用中**|執行 Line-6, 4, 2|

---

## 5. 狀態機（State Machine）深入剖析

### 5.1 狀態機的組成結構

編譯器會將你的 async 方法轉換成類似這樣的結構：

```csharp
// 原始程式碼
public async Task<int> CalculateAsync(int x)
{
    int temp = x * 2;
    await Task.Delay(1000);
    temp = temp + 5;
    await Task.Delay(500);
    return temp + 10;
}

// 原始方法被「掏空」了
// 當你寫下 async 方法時，編譯器實際上會產生類似這樣的程式碼
public Task<int> CalculateAsync(int x)
{
    // 1. 建立狀態機實例
    var stateMachine = new CalculateAsync_StateMachine();
    
    // 2. 初始化狀態機
    stateMachine.x = x;
    stateMachine.state = -1;
    stateMachine.builder = AsyncTaskMethodBuilder<int>.Create();
    
    // 3. 啟動狀態機
    // Start() 內部會呼叫 MoveNext() 🚨
    // MoveNext() 開始執行你的程式碼
    stateMachine.builder.Start(ref stateMachine);
    
    // 4. 立即回傳 Task（還沒完成）
    return stateMachine.builder.Task;
}


// 編譯器產生的狀態機
struct CalculateAsync_StateMachine
{
    // 狀態追蹤
    public int state;
    public AsyncTaskMethodBuilder<int> builder;
    
    // 參數和區域變數
    public int x;
    public int temp;
    
    // 等待器（每個 await 都需要一個）
    private TaskAwaiter delayAwaiter1;
    private TaskAwaiter delayAwaiter2;
    
    // 核心方法
    // 原本 CalculateAsync 方法內容，完全被搬到狀態機的 MoveNext() 裡了！
    public void MoveNext()
    {
        int result;
        try
        {
            switch (state)
            {
                case -1: // 初始狀態
                    // 執行第一段同步程式碼
                    temp = x * 2;
                    
                    // 第一個 await
                    delayAwaiter1 = Task.Delay(1000).GetAwaiter();
                    
                    if (!delayAwaiter1.IsCompleted)
                    {
                        state = 0; // 記住位置：第一個 await
                        delayAwaiter1.OnCompleted(continuation); // 註冊「當 Task 完成時，回呼 MoveNext」🚨
                        return;
                    }
                    goto case 0;
                    
                case 0: // 從第一個 await 恢復
                    delayAwaiter1.GetResult();
                    
                    // 執行兩個 await 之間的程式碼
                    temp = temp + 5;
                    
                    // 第二個 await
                    delayAwaiter2 = Task.Delay(500).GetAwaiter();
                    
                    if (!delayAwaiter2.IsCompleted)
                    {
                        state = 1; // 記住位置：第二個 await
                        delayAwaiter2.OnCompleted(continuation); // 註冊「當 Task 完成時，回呼 MoveNext」🚨
                        return;
                    }
                    goto case 1;
                    
                case 1: // 從第二個 await 恢復
                    delayAwaiter2.GetResult();
                    
                    // 執行最後一段程式碼
                    result = temp + 10;
                    
                    // 設定最終結果
                    state = -2; // 標記為完成
                    builder.SetResult(result);
                    return;
            }
        }
        catch (Exception ex)
        {
            state = -2; // 標記為完成（但是有錯誤）
            builder.SetException(ex);
        }
    }
}
```

狀態機在方法被呼叫時就建立，而不是等到 await 時才建立。

### 5.2 狀態機的四大功能

#### 1. 區域變數的「保險箱」

- **問題**：執行緒離開後，Stack 上的變數會消失
- **解決**：狀態機將變數存在 Heap 上
- **效果**：不同執行緒可以存取相同資料

#### 2. 程式碼的「書籤」

狀態值的意義：

| 狀態值   | 意義               |
| ----- | ---------------- |
| `-1`  | 初始狀態（方法開始）       |
| `0`   | 停在第一個 await      |
| `1`   | 停在第二個 await      |
| `...` | 如果有更多 await，依序遞增 |
| `-2`  | 執行完畢（成功或失敗）      |

#### 3. 「下一步」的聯繫方式

```csharp
// 狀態機持有 TaskAwaiter
// 當 Task 完成時，會呼叫 MoveNext
awaiter.OnCompleted(() => stateMachine.MoveNext());
```

#### 4. 方法本身的狀態管理

- 更新 `Task.Status`
- 設定 `Task.Result`（如果有回傳值）
- 記錄 `Task.Exception`（如果發生錯誤）

### 5.3 為什麼需要狀態機？

|問題|狀態機的解決方案|
|---|---|
|執行緒切換後如何繼續？|state 欄位記住執行位置|
|區域變數會消失？|變數變成狀態機的欄位|
|如何知道任務完成？|TaskAwaiter 註冊回呼|
|錯誤如何傳遞？|透過 builder.SetException|

---

## 6. 錯誤處理與異常管理

### 6.1 await 與 try-catch 的正確配置

#### 正確範例：await 在 try 內

```csharp
public async Task ProcessUserAction()
{
    try 
    {
        // ✅ 正確：將 await 放在 try 裡面
        await OrderCoffeeAsync(); 
        
        Console.WriteLine("3. 交付咖啡給客人。");
    }
    catch (Exception ex) 
    {
        Console.WriteLine($"[錯誤處理] 發生了狀況：{ex.Message}");
    }
}
```

### 6.2 火後不理（Fire and Forget）的問題

#### 問題範例

```csharp
static async Task ProcessData()
{
    Console.WriteLine("Line-3");
    DoHeavyWorkAsync();  // 沒有 await！
    Console.WriteLine("Line-4");
}

static async Task DoHeavyWorkAsync()
{
    Console.WriteLine("Line-5");
    await Task.Delay(1000);
    throw new Exception("糟糕！");  // 這個錯誤會消失
    Console.WriteLine("Line-6");
}
```

**問題分析**：

1. `DoHeavyWorkAsync` 回傳的 Task 無人認領
2. 異常被困在 Task.Exception 屬性裡
3. 沒有 await 或 .Result 來觸發異常拋出
4. 程式可能完全不知道出過錯

#### 解決方案一：延後 await

```csharp
static async Task ProcessData()
{
    Console.WriteLine("Line-3");
    
    // 先存下 Task，但不立刻 await
    Task heavyTask = DoHeavyWorkAsync(); 
    
    // 可以先做其他事
    Console.WriteLine("Line-4"); 
    await Task.Run(() => ReportWork());
    Console.WriteLine("Line-7");
    
    // 最後記得要負責
    await heavyTask; // 如果有錯誤，會在這裡拋出
}
```

#### 解決方案二：內部處理

```csharp
static async Task DoHeavyWorkAsync()
{
    try 
    {
        Console.WriteLine("Line-5");
        await Task.Delay(1000);
        // 可能發生的錯誤
    }
    catch (Exception ex)
    {
        // 自己處理，確保錯誤被記錄
        Console.WriteLine($"[錯誤紀錄]: {ex.Message}");
    }
}
```

### 6.4 異常類型的差異

|存取方式|異常類型|範例程式碼|
|---|---|---|
|`await task`|原始異常|`catch (FileNotFoundException)`|
|`task.Wait()`|AggregateException|`catch (AggregateException)`|
|`task.Result`|AggregateException|`catch (AggregateException)`|
|不存取|無（錯誤遺失）|永遠不會觸發 catch|

---

## 7. 實務範例與情境分析

### 7.1 並行處理範例

```csharp
public async Task ProcessMultipleFilesAsync(string[] filePaths)
{
    // 方法一：依序處理（較慢）
    foreach (var path in filePaths)
    {
        await ProcessFileAsync(path);
    }
    
    // 方法二：並行處理（較快）
    var tasks = filePaths.Select(path => ProcessFileAsync(path));
    await Task.WhenAll(tasks);
    
    // 方法三：限制並行度
    var semaphore = new SemaphoreSlim(3); // 最多 3 個並行
    var tasks = filePaths.Select(async path =>
    {
        await semaphore.WaitAsync();
        try
        {
            await ProcessFileAsync(path);
        }
        finally
        {
            semaphore.Release();
        }
    });
    await Task.WhenAll(tasks);
}
```

---

## 8. 最佳實踐與常見陷阱

### 8.2 常見陷阱

#### 1. async void 的危險

```csharp
// ❌ 危險：async void
public async void ProcessData()
{
    await Task.Delay(1000);
    throw new Exception("這個錯誤會讓程式崩潰！");
}

// ✅ 安全：async Task
public async Task ProcessDataAsync()
{
    await Task.Delay(1000);
    throw new Exception("這個錯誤可以被捕捉");
}
```

**規則**：除了事件處理器，永遠使用 `async Task`

#### 3. 忘記 await

```csharp
// ❌ 忘記 await
public async Task ProcessAsync()
{
    DoSomethingAsync(); // Fire and forget，錯誤會遺失
}

// ✅ 記得 await
public async Task ProcessAsync()
{
    await DoSomethingAsync();
}
```

---

## 總結

非同步程式設計的核心價值在於「等待但不空等」。透過狀態機的機制，.NET 讓我們能用寫同步程式的直覺，寫出高效能的非同步程式。

### 關鍵要點回顧

1. **執行緒的生命週期**：同步起跑 → 遇到等待 → 存檔釋放 → 任務完成 → 恢復執行
2. **狀態機的角色**：保存變數、記錄位置、管理續接
3. **錯誤處理原則**：await 必須在 try 內、避免 fire-and-forget
4. **最佳實踐**：Async all the way、正確使用 ConfigureAwait、支援取消

記住：這套「點餐 → 回櫃台 → 誰有空誰送餐」的邏輯，就是 .NET 達到高效能與高吞吐量的秘密。
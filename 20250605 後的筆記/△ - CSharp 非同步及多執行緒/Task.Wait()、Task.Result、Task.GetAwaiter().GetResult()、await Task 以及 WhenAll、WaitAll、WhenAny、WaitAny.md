---
date: 2026-02-24 15:29
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

## 第一章：同步阻塞 — Task.Wait() 與 Task.Result

在 C# 的非同步程式設計中，`Task.Wait()` 與 `Task.Result` 是用來處理 Task 狀態的兩種方式。它們的核心特性在於會將原本「非同步」的流程轉變為「同步阻塞（Blocking）」的行為。

簡單來說：

- **Task.Wait()** 🛑：這是一個方法，用來等待任務完成。它沒有回傳值（void），主要用於你只需要確保任務執行完畢，但不關心回傳結果的場景。
- **Task.Result** 📥：這是一個屬性，它同樣會等待任務完成，但在完成後會回傳結果。如果你存取這個屬性時任務尚未完成，當前執行緒就會停下來等它。

雖然這兩者看起來很方便，但在 ASP.NET 或 UI 應用程式中使用時需要特別小心，因為它們可能會導致系統資源的浪費，甚至引發程式當機。

### 1.1 基本用法範例

這些方法會將非同步的任務強制轉為「同步」執行，這意味著主執行緒會停下來等待。在這個範例中，我們模擬一個耗時的工作。`Wait()` 用於純粹等待，而 `Result` 則用於獲取回傳值。

```csharp
using System;
using System.Threading.Tasks;

public class Program
{
    public static void Main()
    {
        // 建立一個非同步任務
        Task<string> task = Task.Run(() => {
            Task.Delay(1000).Wait(); // 模擬耗時工作 ⏳
            return "任務完成！";
        });

        Console.WriteLine("正在等待任務...");

        // 🛑 Task.Wait(): 只等待不回傳
        task.Wait(); 
        Console.WriteLine("Wait() 結束了。");

        // 📥 Task.Result: 等待並取得結果
        // 如果執行到這行時 task 還沒完，它也會在這邊阻塞等待
        string message = task.Result; 
        Console.WriteLine($"收到結果: {message}");
    }
}
```

### 1.2 錯誤處理：AggregateException

當使用 `Wait()` 或 `Result` 時，如果任務內部發生錯誤，.NET 為了處理可能同時發生多個錯誤的情況，會將原始錯誤包裝在 `AggregateException` 裡面。

```csharp
try
{
    Task taskWithError = Task.Run(() => {
        throw new InvalidOperationException("自訂的錯誤訊息內容");
    });

    taskWithError.Wait(); // 在這行會拋出異常 💥
}
catch (AggregateException ae)
{
    // 💡 必須透過 InnerException 或 Flatten() 來取得真正的錯誤
    foreach (var ex in ae.InnerExceptions)
    {
        Console.WriteLine($"捕捉到包裝後的錯誤: {ex.Message}");
    }
}
```

這裡要注意，錯誤被包在了 `AggregateException` 之中。這與我們平常使用 `await` 時直接抓到 `InvalidOperationException` 的體驗很不一樣。

---

## 第二章：另一種同步阻塞 — Task.GetAwaiter().GetResult()

`Task.GetAwaiter().GetResult()` 在行為上也非常類似 `Wait()` / `Result`，它同樣會**同步阻塞**當前的執行緒直到任務完成。但在「錯誤處理」的機制上，它與前者有著關鍵的分別。

### 2.1 與 Wait() / Result 的比較

|特性|`Task.Wait()` / `Task.Result`|`Task.GetAwaiter().GetResult()`|
|---|---|---|
|**行為**|同步阻塞 🛑|同步阻塞 🛑|
|**錯誤拋出方式**|拋出 `AggregateException`（包裝過）📦|拋出**原始異常**（未包裝）⚡|
|**主要用途**|一般同步呼叫非同步時使用|編譯器產生 `await` 代碼時的底層實作|

### 2.2 為什麼這很重要？

在進行除錯時，如果錯誤被包在 `AggregateException` 裡，你的 `catch (InvalidOperationException ex)` 可能會抓不到錯誤，因為實際上丟出來的是「裝著錯誤的箱子」，而不是錯誤本身。

但如果你使用 `GetResult()`，它會把箱子拆掉，直接把裡面的 `InvalidOperationException` 丟出來。

舉例來說，假設你在處理一個會丟出 `SqlException` 的非同步資料庫查詢：

- 使用 `Task.Result`：丟出的錯誤型態是 `AggregateException`，`catch (SqlException)` 抓不到。
- 使用 `Task.GetAwaiter().GetResult()`：會把箱子拆掉，直接取得 `SqlException`，讓 `catch` 區塊看起來跟一般的同步程式碼更像。

### 2.3 Task 內部的錯誤永遠是 AggregateException

不論你使用哪種方式取值，在 `Task` 物件內部的 **`Exception` 屬性**中，錯誤**永遠**都會被包裝成一個 `AggregateException` 📦。這是因為 `Task` 的設計初衷之一是為了支援並行運算（Parallel Programming）。當你同時執行多個任務時，可能會有多個錯誤同時發生，因此 .NET 需要一個「容器」來一次裝載所有的錯誤訊息。

當一個非同步任務失敗時，它的狀態會變更為 `Faulted`，並將捕捉到的所有例外狀況存放在 `task.Exception` 這個屬性中。

- **`Task.Result` 或 `Task.Wait()`**：直接把這個 `task.Exception`（也就是那個 `AggregateException` 箱子）整顆丟出來 🧱。
- **`GetResult()` 或 `await`**：它們會執行一個「拆箱（Unwrapping）」的動作 ✂️。它們會檢查箱子裡的 `InnerExceptions` 集合，並抓取**第一個**錯誤，然後直接把它丟出來，讓程式碼看起來就像一般的同步錯誤。

---

## 第三章：非同步等待 — await

與我們先前討論的「同步阻塞」方法（如 `Wait()` 或 `Result`）相比，`await` 是非同步程式設計中最核心的關鍵字。

`await` 的核心特徵是**「非同步等待」**。當程式執行到 `await` 時，它不會卡住目前的執行緒 🧵，而是會將控制權交還給呼叫者，讓執行緒在任務完成前先去處理其他工作。

你可以把 `await` 想像成一個**暫停鍵**：

1. **暫停**：當碰到 `await`，目前的方法會暫時掛起 ⏸️。
2. **釋放**：執行緒被釋放，回執行緒池或 UI 佇列去服務別人。
3. **恢復**：當任務完成時，程式會找一個可用的執行緒（通常是原本那一個，視環境而定）回到剛才暫停的地方繼續往下跑 ⏮️。

### 3.1 await 的錯誤處理特性

對於 `await` 關鍵字來說，它的行為是**「一次只拋出一個錯誤」**。⚡

即使內部其實是一個包含多個任務的容器（例如它內部用了 `Task.WhenAll`），當你對這個回傳的 `Task` 進行 `await` 時，C# 的非同步狀態機會自動幫你執行「拆箱」動作，並只拋出它遇到的**第一個**異常。

- **拋出的異常**：當你執行 `await funcAsync()` 時，如果有錯誤發生，程式會直接跳進 `catch (Exception ex)`。這時的 `ex` 只會是**單一個**原始錯誤（例如 `SqlException`），而不是包裝過的 `AggregateException`。
- **Task 物件內部的狀態**：雖然 `await` 只給了你一個錯誤，但 `funcAsync()` 回傳的那個 `Task` 物件，它內部的 `Exception` 屬性其實可能還是裝著一個 `AggregateException` 📦。

### 3.2 await 的核心職責：拆解並重新拋出異常

**`await` 的作用就是幫你把 Task 裡面包著的 Exception 「拆解」並重新拋出（re-throw）。**

當一個非同步方法發生錯誤時，該異常會被包裝在 `Task` 物件的 `Exception` 屬性中（通常是 `AggregateException`）。

- **如果不 `await`：** 呼叫端只會拿到一個「代表任務」的物件，程式會繼續往下跑。除非你主動去檢查 `task.Exception`，否則主執行緒根本不知道裡面出事了。
- **如果使用 `await`：** 編譯器會自動幫你處理。如果 Task 失敗了，`await` 會將內部的第一個異常拋出來，這時它的行為就**像同步程式碼一樣**。

### 3.3 錯誤會「一路丟回」呼叫方法嗎？

**是的，但前提是「每一層都有 await」。**

在 .NET 的非同步機制中，異常的傳遞（Propagation）依賴於 `await` 鏈。如果你的呼叫堆疊（Call Stack）長這樣：

`MethodA` -> `await MethodB` -> `await MethodC (出錯)`

1. `MethodC` 拋出異常，將其存在 Task 中。
2. `MethodB` 因為 `await` 了 `MethodC`，它會抓到這個異常並重新拋出。
3. `MethodA` 因為 `await` 了 `MethodB`，它也會抓到。

**這就是錯誤「丟回呼叫方法」的機制。** 雖然錯誤會一直往上丟，但如果你在最頂層（例如 Controller Action 或 UI 事件）都沒有寫 `try-catch`，這個錯誤最終會變成 **Unhandled Exception（未處理的異常）**，導致網頁顯示 500 Error、視窗程式直接閃退（Crash），或系統日誌（Log）沒記錄到具體原因。

這就像同步程式（Synchronous code）一樣。`try-catch` 不是為了「捕捉」才寫，而是為了「**處理（Handle）**」。如果不寫，程式雖然能偵測到錯誤，但會因為沒人處理而直接崩潰或回傳 500。

### 3.4 實際情境對照：有 await 但沒 try-catch vs 有 try-catch

**情境 A：有 `await` 但沒 `try-catch`**

```csharp
public async Task<IActionResult> Export()
{
    // 如果發生錯誤，錯誤會丟給 .NET Core Middleware，頁面會跳轉到預設錯誤頁
    await _excelService.GenerateClosedXMLAsync(); 
    return Ok();
}
```

**情境 B：有 `await` 且有 `try-catch`（推薦做法）**

```csharp
public async Task<IActionResult> Export()
{
    try 
    {
        await _excelService.GenerateClosedXMLAsync();
        return Ok();
    }
    catch (Exception ex)
    {
        // 你可以在這裡記錄 Log，或回傳友好的錯誤訊息給使用者
        _logger.LogError(ex, "匯出失敗");
        return BadRequest("系統忙碌中，請稍後再試");
    }
}
```

### 3.5 沒有 await 就抓不到錯誤：「遺失的例外狀況」

如果呼叫一個非同步方法但不使用 `await`，`try-catch` 將**無法**捕捉到 Task 內部的錯誤。這就是「遺失的例外狀況 (Swallowed Exceptions)」👻。

```csharp
try {
    var task = DoWorkAsync(); // 注意這裡沒有 await
} catch (Exception ex) {
    Console.WriteLine("抓到囉！"); // ❌ 永遠不會執行
}
```

**為什麼抓不到？** 當你呼叫一個非同步方法且沒有 `await` 時，程式的運作邏輯如下：

1. **進入 Try 區塊**：執行啟動 Task 的那行程式碼。
2. **任務啟動**：Task 被丟到背景執行（或線程池）。此時，對主程式來說，這行程式碼已經「成功執行完畢」了（它成功地**啟動**了任務）。
3. **離開 Try 區塊**：主程式立刻執行下一行。
4. **錯誤發生**：背景的 Task 噴出錯誤 💥。但此時主程式的執行點早就跳出 `try-catch` 範圍，甚至可能在執行完全不相關的代碼。

這就像是你請快遞去送貨（啟動 Task），快遞出發後，你覺得「寄件」這個動作已經完成了，於是你就轉身去廚房煮飯（執行下一行代碼）。就算快遞在路上爆胎了（發生錯誤），正在廚房煮飯的你也不會立刻感應到。🚚💨

### 3.6 同步錯誤 vs 非同步錯誤

有一種特殊情況是抓得到的，那就是錯誤發生在「非同步任務真正開始之前」。

|錯誤類型|發生時機|`try-catch` 能抓到嗎？(無 await)|
|---|---|---|
|**同步錯誤 (Synchronous)**|在方法內還沒碰到第一個 `await` 或 `Task.Run` 之前就拋出。|**可以** ✅（因為它還在當前的呼叫堆疊中）|
|**非同步錯誤 (Asynchronous)**|任務已經回傳 `Task` 物件，在背景執行時才拋出。|**不行** ❌（除非有 `await`、`Wait()` 或存取 `Result`）|

### 3.7 Task 是一個包裹 — 理解錯誤封裝的核心概念

在傳統的同步程式中，錯誤會沿著「呼叫堆疊 (Call Stack)」一路向上傳遞，直到被捕捉為止。但在非同步的世界裡，規則改變了。我們可以用 **「容器 📦」** 的概念來理解：

當一個非同步方法發生錯誤時，如果它回傳的是 `Task`，這個錯誤並不會立刻「噴」出來，而是會被**封裝**在那個 `Task` 物件裡面。

- **沒有 await**：你只是拿到了包裹，但還沒打開它。主程式繼續跑它的，包裹裡的錯誤就安靜地待在 `Task.Exception` 屬性裡。
- **有 await**：這個動作就像是**「打開包裹」**。`await` 會檢查任務狀態，如果任務失敗了，它才會把裡面的 Exception 重新拋出 (Re-throw)，這時外層的 `try-catch` 才能抓到它。

|特性|同步方法 (Synchronous)|非同步任務 (Task without await)|
|---|---|---|
|**錯誤在哪裡？**|直接在執行緒上拋出 💥|被包在 `Task` 物件內部 📦|
|**誰能抓到？**|呼叫堆疊上的任何 `try-catch`|只有「打開」Task 的人（await / Wait / Result）|
|**沒抓到會怎樣？**|程式立刻崩潰|變成「被遺忘的錯誤」，可能導致程式行為異常但不會立刻當掉|

### 3.8 await 錯誤傳遞的總結

1. **系統會丟回錯誤嗎？** 會，只要你有用 `await`，它就會像傳統錯誤一樣層層向上傳遞。
2. **一定要 `try-catch` 嗎？** 不是為了「捕捉」才寫，而是為了「**處理（Handle）**」。如果不寫，程式雖然能偵測到錯誤，但會因為沒人處理而直接崩潰或回傳 500。
3. **最危險的情況：** 是 `async void`（常用在舊的 WinForms 事件）。這種情況下即使外層寫了 `try-catch` 也攔截不到，會直接讓整個 Process 掛掉。對於 GTS 系統（.NET Framework 4.8），這點尤其重要，因為舊框架對於未處理非同步異常的容忍度有時比 .NET Core 更低。

### 3.9 使用 await 時，錯誤何時被觀察到？

以下面這段程式碼為例：

```csharp
Task allTasks = Task.WhenAll(task1, task2, task3);
await allTasks;
```

- **`Task.WhenAll(...)`** 🏗️：這個方法就像是「建立一個監工專案」。它會立刻回傳一個代表所有任務總合的新 `Task` 物件。即便傳入的任務中已經有人失敗了，這行程式碼本身**不會**拋出異常，它只是把這些任務「包裝」在一起。
- **`await allTasks`** ⚡：這行才是真正的「檢查點」。當程式執行到 `await` 時，它會去觀察這個任務包裹的狀態。如果包裹裡的任務有任何失敗，`await` 就會在這個時間點將異常「拆封」並拋出來。

我們可以用這個比喻來理解：

|程式碼|比喻 💡|
|---|---|
|`Task.WhenAll(...)`|經理把三個員工的報告裝進一個公事包 📦。即便有人沒寫完或寫錯，裝進袋子的動作本身是沒問題的。|
|`await allTasks`|經理打開公事包開始讀報告 ✉️。這時才會發現：「糟糕，有人寫錯了！」並引發後續的錯誤處理。|

---

## 第四章：三種等待方式的總比較

|特性|`Task.Wait()` / `.Result`|`Task.GetAwaiter().GetResult()`|`await`|
|---|---|---|---|
|**等待方式**|同步阻塞（會卡住）🛑|同步阻塞（會卡住）🛑|非同步等待（釋放執行緒）⏳|
|**錯誤呈現**|`AggregateException` 📦|原始異常（單一）⚡|原始異常（單一）⚡|
|**建議使用**|盡量避免 ⚠️|特殊同步需求（如 Main）|**首選方式** ✅|

---

## 第五章：多任務等待 — WaitAll 與 WhenAll

當你需要同時啟動多個任務，並等待它們**全部完成**時，就需要用到 `WaitAll` 或 `WhenAll`。

### 5.1 WaitAll vs WhenAll

- **`Task.WaitAll(task1, task2)`** 🛑：這是一種**同步阻塞**的操作。當程式執行到這一行時，當前的執行緒（Thread）會完全停下來，直到括號內所有的任務都完成、取消或發生錯誤為止，才會移動到下一行。
- **`Task.WhenAll(task1, task2)`** ⏳：這是一個**非同步**的方法。它會立即回傳一個代表「所有任務總合」的新 Task。如果你沒有加上 `await`，程式碼會直接往下衝；如果你加上了 `await`，它會以非同步的方式等待，這意味著執行緒可以先去處理別的事，直到任務都完成了再回來執行下一行。

這種「阻塞」與「非同步等待」的差異，正是避免 UI 介面卡死或提升伺服器吞吐量的關鍵。

### 5.2 Task.WhenAll 的回傳型態

`Task.WhenAll` 的回傳型態會根據你傳入的 Task 種類而有所不同。

#### 如果 Task 有回傳值（Task<T>）

假設你的三個 Task 型態都是 `Task<string>`：

```csharp
Task<string[]> allTasks = Task.WhenAll(task1, task2, task3);
string[] results = await allTasks;
```

- **`allTasks` 的型態**是 **`Task<string[]>`**。`Task.WhenAll` 會將多個個別的任務包裝成一個「大任務」，這個大任務的目標是：等待所有人完成，並把結果收集成一個**陣列**。
- **`results` 的型態**是 **`string[]`**（字串陣列）。當你對 `Task<string[]>` 使用 `await` 時，編譯器會幫你「拆箱」。`await` 結束後，非同步的任務已經完成，你拿到的就是實實在在的資料陣列。

#### 如果 Task 沒有回傳值（Task）

假設你的 Task 只是單純執行動作（例如存檔、發送 Email），型態是 `Task`：

- **`allTasks` 的型態**是 **`Task`**。因為沒有回傳值，它單純代表「這三件事都做完」這個狀態。
- 如果你寫 `var results = await allTasks;` 而 `allTasks` 只是 `Task`，這在 C# 中是不合法的，因為 `Task`（非泛型）的 `await` 結果是 `void`。

#### 型態對照表

|情況|`allTasks` 的型態（Before await）|`results` 的型態（After await）|
|---|---|---|
|**傳入多個 `Task<int>`**|`Task<int[]>`|`int[]`|
|**傳入多個 `Task<Customer>`**|`Task<Customer[]>`|`Customer[]`|
|**傳入多個 `Task`（無回傳）**|`Task`|無（不能賦值給變量）|
|**混合不同型態（不建議）**|`Task<object[]>` 或 `Task`|`object[]` 或 無|

### 5.3 AggregateException — 多任務的錯誤容器

當我們使用 `Task.WaitAll()` 同時等待多個任務時，.NET 的設計目標是確保我們不會漏掉任何一個執行過程中的失敗資訊。因此，它使用 `AggregateException` 📦 作為一個容器來「匯總」這些錯誤。

這個容器裡有一個關鍵屬性叫做 `InnerExceptions`（注意是複數），它是一個集合，專門收集所有在任務執行期間拋出的異常。

舉例來說，如果我們啟動了 3 個任務 🧵，其中有 2 個任務因為不同的原因失敗了 ❌，`InnerExceptions` 集合裡面就會包含 **2 個**錯誤物件。不論同時有多少個任務失敗，它都會把這些錯誤一個一個放進 `InnerExceptions` 集合裡。這樣一來，開發者就能透過迴圈遍歷這個集合，掌握每一個失敗任務的細節。

### 5.4 AggregateException vs await 處理方式

|方式|錯誤拋出行為|
|---|---|
|**`Task.Wait()` / `Result`**|將所有錯誤包裝在 `AggregateException` 裡拋出 📦|
|**`await`**|僅拋出**第一個**捕獲到的原始錯誤（Unwrapped）⚡|

這種差異在除錯時非常關鍵。如果你使用 `await`，程式碼看起來比較簡潔，但你預設只能抓到「第一個」錯誤訊息，這可能會讓你忽略掉其他同時發生的問題。

### 5.5 使用 Task.WhenAll 搭配 await — 兩全其美

如果需要在不鎖死執行緒的情況下，依然能拿到所有的錯誤資訊，關鍵角色就是 **`Task.WhenAll`**。🌐

`Task.WhenAll` 與 `Task.WaitAll` 的功能相似，都是等待多個任務完成，但它們有一個核心差異：

- **`Task.WaitAll`**：是同步阻塞（Blocking），會卡住執行緒直到所有任務結束。🛑
- **`Task.WhenAll`**：會回傳一個**新的 Task**。你可以使用 `await` 來等待這個 Task，這讓執行緒在等待期間可以去處理其他事情，從而避免鎖死（Deadlock）。⚡

當我們 `await Task.WhenAll(...)` 時，如果有多個任務失敗，`await` 確實只會拋出**第一個**捕獲到的異常。但是，重點在於：雖然 `await` 只給了你一個錯誤，但那個由 `WhenAll` 產生的**「總合 Task」**，其實內部仍然完整保存了所有的錯誤訊息。我們可以透過 `allTasks.Exception`（型態是 `AggregateException`）來取得它。

### 5.6 捕捉所有錯誤的完整範例

為了在 `catch` 區塊中存取任務狀態，我們需要先在 `try` 外部宣告變數。以下是常用的結構：

```csharp
Task allTasks = null; // 1. 先在 try 外部宣告變數

try 
{
    allTasks = Task.WhenAll(t1, t2, t3); // 2. 賦值
    await allTasks; // 3. 非同步等待（發生錯誤會跳到 catch）
}
catch (Exception) 
{
    // 4. 因為 allTasks 在 try 外宣告，這裡可以順利存取
    if (allTasks?.Exception != null) 
    {
        // 使用 foreach 遍歷 InnerExceptions，取得每一個錯誤
        // 建議加上 .Flatten() 把嵌套的錯誤攤平
        foreach (var ex in allTasks.Exception.Flatten().InnerExceptions) 
        {
            Console.WriteLine($"捕捉到詳細錯誤：{ex.Message}");
        }
    }
}
```

### 5.7 關於 Flatten() — 處理巢狀的 AggregateException

在處理複雜的非同步情境時，有時候錯誤會「層層包裝」（例如一個 `AggregateException` 裡面又包了另一個 `AggregateException`）。這在「任務 A」執行到一半又啟動了「子任務 B」，且這兩個任務都發生錯誤的時候會出現。

- **直接使用 `InnerExceptions`**：你可能只會抓到第二層的箱子，而不是最底層的錯誤。
- **使用 `Flatten()`** 🔨：它會把所有層級的錯誤全部抽出來，攤平成一個單層的集合，讓你一次看清楚所有的原始異常。

```csharp
// 攤平所有嵌套的 AggregateException
allTasks.Exception.Flatten().InnerExceptions
```

### 5.8 await 拋出哪一個錯誤？— 參數順序決定

當多個任務同時失敗時，`await` 必須選擇一個來拋出。它選擇的標準**不是**誰「先」噴出錯誤，而是誰在 `WhenAll(task1, task2, task3)` 的括號裡面**排在最前面**。

|發生錯誤的任務|`allTasks.Exception`（容器內容）|`await` 拋出的錯誤|
|---|---|---|
|**只有 Task 3**|`AggregateException`（內含 1 個錯誤）|Task 3 的錯誤|
|**Task 2 與 Task 3**|`AggregateException`（內含 2 個錯誤）|Task 2 的錯誤|
|**全部成功**|`null`|無錯誤|

舉例來說，如果你同時啟動了兩個任務：

- `taskA`：需要執行 10 秒，最後失敗了。
- `taskB`：需要執行 1 秒，最後也失敗了。

如果你寫成 `await Task.WhenAll(taskA, taskB);`，雖然 `taskB` 在 1 秒時就先出錯了，但當 10 秒後整個 `WhenAll` 結束時，`await` 拋出的會是 `taskA` 的錯誤訊息，因為 `taskA` 在參數列表中排在前面（Index 0）。

只要這群任務中有**任何一個**（不管是第幾個）發生錯誤，`WhenAll` 回傳的總合 Task，其內部的 `.Exception` 屬性**一定會**是一個 `AggregateException` 📦。

---

## 第六章：多任務競賽 — WhenAny 與 WaitAny

與 `WhenAll` / `WaitAll` 等待「全部完成」不同，`WhenAny` / `WaitAny` 的目標是**「只要其中一個完成」**。這在處理「逾時控制 (Timeout)」或「多來源競爭 (Racing)」時非常強大。

### 6.1 WhenAny vs WaitAny

|特性|`Task.WaitAny` 🛑|`Task.WhenAny` ⏳|
|---|---|---|
|**執行方式**|同步阻塞（Blocking）|非同步等待（Asynchronous）|
|**回傳內容**|完成任務的**索引 (Index)**，型態為 `int`|**已完成的 Task 物件**本身，型態為 `Task<Task>`|
|**使用情境**|舊版同步程式碼或 Main 方法|現代非同步開發（Controller, API）|

當你呼叫 `WaitAny` 時，執行緒會卡在原地，直到某個任務傳回完成訊號，它會告訴你是「第幾個」任務贏了。而 `WhenAny` 則是回傳一個「正在等待完成任務」的包裹，當你 `await` 它時，它會把那個「跑最快的 Task 物件」直接交給你。

### 6.2 WhenAny 的結束時機

`Task.WhenAny` 的核心就是**「誰先結束就回傳誰」**。在 .NET 的定義中，一個任務只要進入了「最終狀態」（Terminal State），不論結果是成功、發生錯誤還是被取消，都算作「結束」。🏁

這就像是一場賽跑，裁判只負責記錄誰第一個衝過終點線。即使第一名跨過線後立刻跌倒受傷（發生錯誤），他依然是「第一個抵達」的人。🏎️

|狀態 (Status)|意義|`WhenAny` 會回傳嗎？|
|---|---|---|
|**RanToCompletion** ✅|任務成功執行完畢|**會**，它是第一個跑完的|
|**Faulted** ⚠️|任務執行中發生異常|**會**，它是第一個「失敗」的|
|**Canceled** 🚫|任務被取消了|**會**，它是第一個「被叫停」的|

即使第一個任務噴錯了，`WhenAny` 也會立刻把它丟回來。它在乎的是**時間**，而不是**品質**。

### 6.3 實作逾時控制 (Timeout Pattern)

這是 `Task.WhenAny` 最常見的用途之一。實作的邏輯就像是一場**競賽** 🏎️：我們讓「真正的任務」與「計時器任務」比賽，看誰先抵達終點。

```csharp
// 1. 啟動真正的非同步任務 (例如呼叫 API)
Task serviceTask = CallRemoteApiAsync();

// 2. 啟動一個計時器任務 (例如設定 5 秒)
Task delayTask = Task.Delay(5000);

// 3. 讓兩者比賽，看誰先完成
Task completedTask = await Task.WhenAny(serviceTask, delayTask);

// 4. 判斷誰是贏家 🏆
if (completedTask == delayTask)
{
    // 計時器先跑完 = 逾時了 🛑
}
else
{
    // 真正的任務先完成 = 正常處理結果 ✅
}
```

`Task.WaitAny` 也能達成相同目的，但因為它會**阻塞執行緒**，在 .NET Core 的 Web 應用中，如果伺服器端的執行緒全部都被 `WaitAny` 叫去「發呆等待」了，對系統的**吞吐量**（同時能服務的人數）會造成嚴重影響。使用 `await Task.WhenAny` 的方式不會阻塞執行緒，當任務在比賽時，執行緒可以放心地去處理其他使用者的請求。

### 6.4 WhenAny 的「拆箱」— Task<Task> 結構

在 `WhenAny` 中，你會遇到一個有趣的型別：`Task<Task>`。這就像是一個「大包裹」裡面裝著一個「小包裹」。

1. **第一層 await**：當你 `await Task.WhenAny(...)` 時，你是在等待「比賽結束」。這個動作通常**不會拋出異常**，即使比賽的任務失敗了，因為「比賽結束」這件事本身是成功的。
2. **第二層拆箱**：你拿到了那個贏家任務（小包裹）後，必須再次「拆開」它才能得到真正的結果或看到錯誤。

```csharp
Task<string> task1 = DoWork1Async();
Task<string> task2 = DoWork2Async();

// 第一層：等待比賽結束，拿到贏家（Task<Task> 的拆箱）
Task<string> winnerTask = await Task.WhenAny(task1, task2);

try 
{
    // 第二層：對贏家進行 await，這時才會拿到 string 結果或噴出 Exception
    string result = await winnerTask; 
}
catch (Exception ex) 
{
    // 如果贏家任務失敗了，錯誤會在這裡被捕捉
}
```

### 6.5 WhenAny 的錯誤處理 — 取得單一與所有錯誤

當比賽結束時，我們不僅想知道誰贏了，如果有人「跌倒」了，我們也想收集所有的受傷報告。

```csharp
public async Task HandleErrorsWithWhenAny()
{
    var t1 = Task.Run(() => { throw new Exception("錯誤 A"); });
    var t2 = Task.Run(async () => { await Task.Delay(100); throw new Exception("錯誤 B"); });
    var tasks = new List<Task> { t1, t2 };

    // 1. 等待其中一個結束（這行不會噴錯）
    Task winnerTask = await Task.WhenAny(tasks);

    try
    {
        // 2. 透過再次 await 贏家，拆開包裹看它是成功還是失敗
        await winnerTask; 
    }
    catch (Exception ex)
    {
        Console.WriteLine($"贏家任務噴錯了：{ex.Message}");
        
        // 3. 獲取「所有」任務中的錯誤 📑
        foreach (var t in tasks)
        {
            if (t.IsFaulted && t.Exception != null)
            {
                // Task 的錯誤都包在 AggregateException 裡
                foreach (var inner in t.Exception.Flatten().InnerExceptions)
                {
                    Console.WriteLine($"收集到任務錯誤：{inner.Message}");
                }
            }
        }
    }
}
```

需要注意的是，`WhenAny` 結束時，其他還在執行的任務並**不會自動停止**。它們會繼續在背景執行，直到完成或出錯。如果你想在 `WhenAny` 結束後取得「所有」任務的錯誤，就必須另外處理。

### 6.6 WaitAny 的完整範例 — 取值與錯誤處理

`Task.WaitAny` 是一個同步阻塞方法。它與 `WhenAny` 最大的不同在於，它回傳的是已完成任務在陣列中的**索引值 (Index)** 🔢。

```csharp
// 1. 建立一組任務
var task1 = Task.Run(() => { Thread.Sleep(2000); return "結果 A"; });
var task2 = Task.Run<string>(() => { throw new InvalidOperationException("發生錯誤 B"); });

var tasks = new[] { task1, task2 };

// 🛑 這裡會阻塞執行緒，直到其中一個結束。回傳的是陣列索引（0, 1, 2...）
int finishedIndex = Task.WaitAny(tasks);

// 2. 取得該完成的任務物件
var completedTask = tasks[finishedIndex];

// 3. 檢查狀態並處理結果或錯誤
if (completedTask.IsFaulted) 
{
    // 💡 注意：WaitAny 本身不會拋出任務內部的異常
    // 異常被包裝在 Exception 屬性中 (AggregateException)
    Console.WriteLine($"任務 {finishedIndex} 失敗：{completedTask.Exception?.InnerException?.Message}");
}
else 
{
    // 取得回傳值 (因為已經完成，這行不會再阻塞)
    Console.WriteLine($"任務 {finishedIndex} 成功：{completedTask.Result}");
}
```

注意到一個細節：雖然 `task2` 拋出了異常，但 `Task.WaitAny(tasks)` 這行程式碼本身**不會拋出異常**。它只是安靜地回傳索引值。真正的異常要等到我們去檢查 `completedTask.Exception` 或是強行存取 `completedTask.Result` 時才會浮現 🕵️。

---

## 第七章：Task 的狀態屬性與 TaskStatus 列舉

`Task` 物件除了透過 `try-catch` 來處理錯誤之外，還提供了幾個非常重要的狀態屬性，讓你可以透過「狀態檢查」來判斷任務的執行結果。這些屬性主要來自 `Task` 的**狀態機（State Machine）**機制。

### 7.1 核心狀態屬性 (Status Properties)

|屬性名稱|型態|說明|
|---|---|---|
|**`IsCompleted`**|`bool`|**最常用。** 不論任務是成功、失敗、還是被取消，只要任務「結束了」，它就是 `true`。|
|**`IsFaulted`**|`bool`|當任務因為**未處理的異常（Exception）**而終止時為 `true`。|
|**`IsCanceled`**|`bool`|當任務因為 **CancellationToken** 被取消而終止時為 `true`。|
|**`Status`**|`Enum`|回傳 `TaskStatus` 列舉，可以更細節地知道它是 `Running`、`WaitingForActivation` 還是 `RanToCompletion`。|

### 7.2 三個 bool 屬性的關係

這三個 `bool` 屬性其實是有重疊與互斥關係的。當 `IsCompleted` 為 `true` 時，代表任務進入了終點站，但終點站有三種可能：

1. **成功：** `IsFaulted` = false, `IsCanceled` = false（這在 `Status` 裡叫 `RanToCompletion`）。
2. **失敗：** `IsFaulted` = true。
3. **取消：** `IsCanceled` = true。

### 7.3 關於 Exception 屬性

當 `IsFaulted` 為 `true` 時，你可以存取 `task.Exception`。

- 它的型態永遠是 **`AggregateException`**。
- 這是因為一個 Task 可能會包含多個子任務，所以它用一個「容器」把所有錯誤包起來。
- 通常我們會用 `task.Exception.Flatten()` 來攤平巢狀錯誤，方便記錄 Log。

### 7.4 實務應用：透過狀態檢查處理結果

假設你在處理 ClosedXML 產製報表，不想讓整個系統噴 Error，而是想根據狀態給提示：

```csharp
Task exportTask = _excelService.GenerateReportAsync();

// 假設我們不用 await 擋住，而是晚點回來檢查
if (exportTask.IsCompleted) 
{
    if (exportTask.IsFaulted) 
    {
        // 這裡可以抓到具體的 Exception，而不用寫 try-catch 塊
        var error = exportTask.Exception?.InnerException;
        _logger.LogError(error, "報表產製失敗");
    }
    else if (exportTask.IsCanceled)
    {
        _logger.LogWarning("使用者取消了下載");
    }
    else 
    {
        _logger.LogInformation("成功完成！");
    }
}
```

### 7.5 狀態檢查 vs. Try-Catch 的使用時機

- **使用 `try-catch` + `await`：** 是最直覺、最像同步程式寫法的方式，適合大多數商業邏輯。
- **使用 `IsFaulted` 檢查：** 適合用在「不想要中斷程式執行」或是「批次處理多個 Task」的情境（例如：先發出一堆 Request，最後再檢查哪些失敗了）。

### 7.6 TaskStatus 列舉的完整生命週期

`TaskStatus` 是一個**列舉 (Enum)**，用來精細地描述 Task 目前處於生命週期的哪一個階段。你可以把 Task 想像成一個**工單**，從建立、排隊、執行到結案，都有對應的狀態。

#### 階段一：準備與排隊 (Preparing)

- **`Created`**：Task 已被初始化，但還沒呼叫 `Start()`。通常手動 `new Task()` 才會看到，如果是 `Task.Run` 會直接跳過這步。
- **`WaitingForActivation`**：Task 正在等待 .NET 內部的排程機制激活，或者正在等待它所依賴的其他 Task 完成。
- **`WaitingToRun`**：Task 已經由排程器（Scheduler）接手，正在排隊等待 CPU 執行。

#### 階段二：執行中 (In Progress)

- **`Running`**：Task 正在執行中，程式碼正在跑。
- **`WaitingForChildrenToComplete`**：主要任務已跑完，但它開出的「子任務」（Attached Child Tasks）還在跑，所以它必須等小孩都回家才能結案。

#### 階段三：最終狀態 (Final States)

一旦進入以下三種狀態，`Task.IsCompleted` 就會變成 `true`。

- **`RanToCompletion`**：**最理想的狀態。** 任務成功執行完畢，沒有報錯，也沒有被取消。
- **`Canceled`**：任務因為 `CancellationToken` 被觸發而安全撤銷。
- **`Faulted`**：任務因為發生未處理的異常（Exception）而崩潰。

### 7.7 實務對照表

|狀態名稱|你的程式碼在幹嘛？|備註|
|---|---|---|
|**`Running`**|`await _service.GetDataAsync();` 正在執行 SQL 或呼叫 API。|執行中。|
|**`RanToCompletion`**|資料抓完了，畫面正常顯示。|這是 `IsCompleted` 且沒有錯誤的狀態。|
|**`Faulted`**|`ClosedXML` 存檔時權限不足，拋出了 `IOException`。|此時 `IsFaulted` 為 `true`。|
|**`Canceled`**|使用者按了「取消匯出」，觸發了 `CancellationToken`。|此時 `IsCanceled` 為 `true`。|

### 7.8 如何在程式碼中使用 TaskStatus？

通常我們不會直接判斷 `TaskStatus.Running`，因為狀態變化很快。但在 **Debug 偵錯**或**單元測試**時，這非常有幫助：

```csharp
var task = DoSomethingAsync();

// 檢查任務是否還在跑
if (task.Status == TaskStatus.Running) 
{
    // 執行某些邏輯...
}

// 或是更嚴謹的判斷成功
if (task.Status == TaskStatus.RanToCompletion)
{
    Console.WriteLine("完美落地！");
}
```

### 7.9 小提醒：.NET Framework 4.8 的差異

在 .NET Framework 4.8 中，如果你使用較舊的 `Task.Factory.StartNew` 而不是 `Task.Run`，你會更頻繁地看到 `WaitingForChildrenToComplete`，因為 `StartNew` 預設支持子任務附加（Parent/Child Tasks），而 `Task.Run` 會自動幫你把任務「攤平」。

---

## 第八章：全景總覽 — 所有等待方式一次比較

### 8.1 單任務等待方式

|特性|`Task.Wait()` / `.Result`|`GetAwaiter().GetResult()`|`await`|
|---|---|---|---|
|**等待方式**|同步阻塞 🛑|同步阻塞 🛑|非同步等待 ⏳|
|**錯誤呈現**|`AggregateException` 📦|原始異常 ⚡|原始異常 ⚡|
|**建議使用**|盡量避免 ⚠️|特殊同步需求|**首選方式** ✅|

### 8.2 多任務 — 等待全部完成

|特性|`Task.WaitAll` 🛑|`Task.WhenAll` ⏳|
|---|---|---|
|**等待方式**|同步阻塞|非同步（搭配 await）|
|**錯誤呈現**|`AggregateException`（包含所有錯誤）|`await` 只拋出第一個，但可透過 `.Exception` 取得全部|
|**取得所有錯誤**|`catch (AggregateException)`|`allTasks.Exception.Flatten().InnerExceptions`|

### 8.3 多任務 — 等待任一完成

|特性|`Task.WaitAny` 🛑|`Task.WhenAny` ⏳|
|---|---|---|
|**等待方式**|同步阻塞|非同步（搭配 await）|
|**回傳內容**|已完成任務的**索引 (int)**|**已完成的 Task 物件**|
|**本身是否拋出異常**|否|否（需再次 await 贏家任務）|
|**取得錯誤**|`tasks[index].Exception`|再次 `await winnerTask`|

### 8.4 WhenAny / WaitAny 的共同特性

- 不管任務是成功、失敗還是被取消，只要「結束」就算完成。
- 本身**不會**拋出任務內部的異常，必須另外檢查或 await。
- 其他還在執行的任務**不會自動停止**，會繼續在背景執行。

### 8.5 Task 狀態屬性速查

|屬性|型態|何時為 true|
|---|---|---|
|`IsCompleted`|`bool`|任務結束（不論成功、失敗、取消）|
|`IsFaulted`|`bool`|任務因異常而終止|
|`IsCanceled`|`bool`|任務因 CancellationToken 而取消|
|`Status`|`TaskStatus`|提供更精細的生命週期階段|
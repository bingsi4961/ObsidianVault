---
date: 2025-08-06 11:55
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

# C# Async/Await 重要觀念筆記

## 1. 非同步處理的三大風險

使用非同步時需要特別注意以下問題：

### 執行比背景執行緒提早結束

- 主程式可能在背景非同步操作完成前就結束
- 特別是在 Console 應用程式中，`Main` 方法結束會終止整個程式

### 無法捕捉例外

- 背景執行的例外不會自動傳播到呼叫端
- Fire-and-Forget 的非同步操作例外容易遺失

### 資源物件被提早釋放

- 物件可能在非同步操作完成前被 GC 回收
- `using` 語句可能在非同步操作完成前就釋放資源

## 2. using 與 await 的互動機制

### 正確的資源管理

```csharp
using (var resource = new SomeResource())
{
    await SomeAsyncOperation(); // using 會等待這個 await 完成
    // 後續程式碼...
} // 資源在這裡才被釋放
```

### 關鍵行為

- `using` 中有 `await` 時，會等待 `await task` 完全完成
- 執行完 `await task` 後面的所有流程
- 最後才回收 `using` 中的物件
- 確保資源在非同步操作期間不會被提早釋放

## 3. 例外處理的層級差異

### 背景執行的例外處理原則

```csharp
_ = Task.Run(async () => {
    try
    {
        // 背景執行的程式碼
        await SomeAsyncOperation();
    }
    catch (Exception ex)
    {
        // 必須在 Task.Run 內部處理例外
        // 外層的 try-catch 無法捕獲這裡的例外
        HandleException(ex);
    }
});
```

### 為什麼需要內部例外處理

- 背景執行的例外不會被外層捕獲
- Fire-and-Forget 操作的例外會被吞沒
- 需要在 `Task.Run` 內部建立完整的例外處理機制

## 4. Task.Run 永遠不會自動等待

**核心概念**：`Task.Run()` 在任何情況下都會立即回傳 Task 物件，不會等待委派函式完成。

## 5. Task.Run 的基本行為

### 所有這些寫法都不會等待：

```csharp
// ❌ 不會等待 - 立即回傳 Task
Task.Run(() => ProcessSoaBomInBackground(unProcessPartsLists));

// ❌ 不會等待 - 立即回傳 Task  
Task.Run(() => { 
    ProcessSoaBomInBackground(unProcessPartsLists); 
});

// ❌ 不會等待 - 立即回傳 Task
Task.Run(() => {
    return ProcessSoaBomInBackground(unProcessPartsLists);
});

// ❌ 不會等待 - 立即回傳 Task
Task.Run(async () => { 
    await ProcessSoaBomInBackground(unProcessPartsLists); 
});
```

### 會等待的正確寫法：

```csharp
// ✅ 會等待 - 明確等待 Task 完成
var task = Task.Run(() => ProcessSoaBomInBackground(unProcessPartsLists));
task.Wait();

// ✅ 會等待 - 使用 GetAwaiter().GetResult()
Task.Run(async () => { 
    await ProcessSoaBomInBackground(unProcessPartsLists); 
}).GetAwaiter().GetResult();

// ✅ 會等待 - 直接等待非同步方法
ProcessSoaBomInBackground(unProcessPartsLists).GetAwaiter().GetResult();

// ✅ 會等待 - 在 async 方法中使用 await
await Task.Run(() => ProcessSoaBomInBackground(unProcessPartsLists));
```

## 6. Console 應用程式的特殊問題

### 問題根源

```csharp
static void Main(string[] args)
{
    // 啟動背景 Task，但沒有等待
    Task.Run(() => SomeAsyncMethod());
    
    // Main 方法結束，整個 Console 應用程式終止
    // 背景 Task 被強制停止
}
```

### 執行順序

1. `Task.Run()` 立即回傳 Task 物件
2. 主執行緒繼續執行
3. `Main()` 方法結束
4. Console 應用程式終止
5. **所有背景執行緒被強制停止**

## 7. Lambda 表達式語法的真相

### Expression Body vs Statement Body

```csharp
// Expression Body - 自動 return
() => ProcessMethod()
// 編譯器轉換為：
() => { return ProcessMethod(); }

// Statement Body - 需要明確 return
() => { 
    ProcessMethod(); // 沒有 return，回傳 void
}
```

### 重要澄清：返回值類型不影響 Task.Run 的等待行為

```csharp
// 這兩種寫法在等待行為上完全相同 - 都不會等待
Task.Run(() => ProcessMethod());           // 返回 Task
Task.Run(() => { ProcessMethod(); });      // 返回 void

// Task.Run 永遠立即回傳，不管委派函式返回什麼
```

## 8. 委派中的控制流問題

### 錯誤寫法：遺棄非同步方法

```csharp
Task.Run(() => { 
    serviceClient.ProcessSoaBomServiceAsync(unProcessPartsLists); // 沒有 await
    // 其他程式碼立即執行
});
```

**問題**：

- `ProcessSoaBomServiceAsync` 回傳的 Task 被忽略
- 委派立即完成，但非同步方法可能還在執行
- 非同步方法可能被提早終止

### 正確寫法：等待非同步方法完成

```csharp
Task.Run(async () => { 
    await serviceClient.ProcessSoaBomServiceAsync(unProcessPartsLists); // 有 await
    // 等待非同步方法完全結束後才執行
});
```

**但是**：<mark style="background: #FFF3A3A6;">Task.Run 本身仍然不會等待這個 async lambda 完成！</mark>


## 10. 重點摘要

### 核心原則

- **Task.Run 永遠立即回傳**，不會等待委派執行完成
- **Console 應用程式在主執行緒結束後立即終止**
- **必須明確等待 Task 才能確保背景工作完成**

### 記住的口訣

- <mark style="background: #FFF3A3A6;">Task.Run 不等待：永遠立即回傳 Task 物件</mark>
- **Console 會終止**：主執行緒結束後整個應用程式結束
- **背景會中斷**：未完成的背景任務會被強制停止
- **必須明確等**：使用 Wait()、GetAwaiter().GetResult() 或 await

### Web vs Console 差異

- <mark style="background: #FFF3A3A6;">Web 應用程式：伺服器持續運行，背景 Task 有機會完成</mark>
- <mark style="background: #FFF3A3A6;">Console 應用程式：主執行緒結束後立即終止，背景 Task 被強制停止</mark>
- <mark style="background: #FFF3A3A6;">Task.Run 行為完全相同：在兩種環境中都立即回傳，不會等待</mark>
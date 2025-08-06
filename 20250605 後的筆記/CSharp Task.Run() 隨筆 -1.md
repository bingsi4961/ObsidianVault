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

## 4. Task.Run 委派中的控制流問題

### 錯誤寫法：非同步方法沒有 await

```csharp
Task.Run(() => { 
    serviceClient.ProcessSoaBomServiceAsync(unProcessPartsLists); // 沒有 await
    // 其他程式碼...
});
```

**控制流問題**：

1. 呼叫 `ProcessSoaBomServiceAsync`，返回 `Task`
2. **控制流立即跳出**非同步方法，繼續執行後面流程
3. 委派快速完成，`Task.Run` 結束
4. 但是 `ProcessSoaBomServiceAsync` 的實際工作可能還在進行
5. **結果**：非同步方法被「遺棄」，可能被提早終止

### 正確寫法：非同步方法加上 await

```csharp
Task.Run(async () => { 
    await serviceClient.ProcessSoaBomServiceAsync(unProcessPartsLists); // 有 await
    // 其他程式碼...
});
```

**控制流修正**：

1. 呼叫 `ProcessSoaBomServiceAsync`
2. **控制流暫時跳出** `Task.Run`，但委派尚未完成
3. 等待非同步方法完全結束
4. **控制流回到** `Task.Run`，繼續執行後續流程
5. 委派真正完成，`Task.Run` 結束

## 5. Task.Run 的委派返回值機制

### Lambda 表達式的兩種語法

#### Expression Body（表達式主體）- 自動 return

```csharp
() => ProcessSoaBomInBackground(unProcessPartsLists)
```

**編譯器自動轉換為**：

```csharp
() => {
    return ProcessSoaBomInBackground(unProcessPartsLists); // 自動加 return
}
```

#### Statement Body（語句主體）- 需要明確 return

```csharp
() => { 
    ProcessSoaBomInBackground(unProcessPartsLists); // 沒有 return
    // 隱含 return void
}
```

### Task.Run 的行為差異

#### 委派返回 Task - Task.Run 會等待

```csharp
// 返回 Task，Task.Run 會等待這個 Task 完成
Task.Run(() => ProcessSoaBomInBackground(unProcessPartsLists));

// 等同於
Task.Run(() => {
    return ProcessSoaBomInBackground(unProcessPartsLists);
});
```

**Task.Run 使用重載**：`Task.Run(Func<Task> function)`

#### 委派返回 void - Task.Run 立即完成

```csharp
// 返回 void，Task.Run 立即完成
Task.Run(() => { 
    ProcessSoaBomInBackground(unProcessPartsLists); // Task 被忽略
});
```

**Task.Run 使用重載**：`Task.Run(Action action)`

## 6. 實戰建議

### 背景執行的最佳實踐

```csharp
// 方法 A：使用 Expression Body
_ = Task.Run(() => ProcessSoaBomInBackground(unProcessPartsLists));

// 方法 B：使用 Statement Body 但明確 return
_ = Task.Run(() => {
    return ProcessSoaBomInBackground(unProcessPartsLists);
});

// 方法 C：使用 async/await
_ = Task.Run(async () => {
    try
    {
        using (var serviceClient = new SoaBomServiceClient())
        {
            await serviceClient.ProcessSoaBomServiceAsync(unProcessPartsLists);
        }
    }
    catch (Exception ex)
    {
        HandleException(ex);
    }
});
```

### 記住的口訣

- **有大括號 `{}`**：需要明確 `return`，否則返回 `void`
- **沒有大括號**：編譯器自動加 `return`
- **委派返回 Task**：`Task.Run` 會等待
- **委派返回 void**：`Task.Run` 立即完成
- **背景例外**：必須在 `Task.Run` 內部處理
---
date: 2025-10-28 17:42
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
## 問題程式碼

```csharp
public ActionResult GetData()
{ 
    var result = GetDataAsync().Result;  // ← 使用 .Result 阻塞
    return View(result);
}

private async Task<Data> GetDataAsync()
{ 
    await Task.Delay(1000).ConfigureAwait(false); 
    await Task.Delay(500);  // ← 沒有 ConfigureAwait(false)
    return new Data();
}
```

---

## 死鎖發生原因

### 1. 阻塞式等待

- `GetData()` 使用 `.Result` 阻塞當前執行緒
- 等待非同步操作完成

### 2. ASP.NET 同步上下文機制

- ASP.NET Framework 有特殊的 SynchronizationContext
- `await` 完成後預設會嘗試回到原始同步上下文繼續執行

### 3. ConfigureAwait 作用範圍

- `ConfigureAwait(false)` **只影響該次 await 之後的程式碼**
- 不影響後續的 await

---

## 執行流程分析

```csharp
private async Task<Data> GetDataAsync()
{ 
    // ▼ 階段 1：在原始 ASP.NET request context
    
    await Task.Delay(1000).ConfigureAwait(false);
    
    // ▼ 階段 2：已脫離原始上下文，在執行緒池執行緒
    
    await Task.Delay(500);  // 預設 ConfigureAwait(true)
    
    // ▼ 階段 3：嘗試回到原始同步上下文
    //   但原始上下文被 .Result 阻塞 → 死鎖！
    
    return new Data();
}
```

### 死鎖原因詳解

1. 第一個 `await` + `ConfigureAwait(false)` → 成功脫離原始上下文
2. 第二個 `await` 沒有 `ConfigureAwait(false)` → 嘗試重新捕獲並回到原始上下文
3. 原始上下文被 `.Result` 阻塞 → 無法進入 → **死鎖**

---

## 不同框架的行為差異

### ASP.NET Framework (如 GTS 系統 .NET Framework 4.8)

- ✗ **會死鎖**
- 有 SynchronizationContext

### ASP.NET Core (如 Portal 系統 .NET Core 3.1)

- ✓ **不會死鎖**
- 預設沒有 SynchronizationContext

---

## 解決方案

### ✅ 方案 1：全部改用 async/await（最佳實踐）

```csharp
public async Task<ActionResult> GetData()
{ 
    var result = await GetDataAsync();
    return View(result);
}

private async Task<Data> GetDataAsync()
{ 
    await Task.Delay(1000);
    await Task.Delay(500);
    return new Data();
}
```

**優點：**

- 完全避免死鎖
- 不阻塞執行緒
- 符合最佳實踐

---

### ⚠️ 方案 2：所有 await 都加 ConfigureAwait(false)

```csharp
private async Task<Data> GetDataAsync()
{ 
    await Task.Delay(1000).ConfigureAwait(false);
    await Task.Delay(500).ConfigureAwait(false);  // 加上這個
    return new Data();
}
```

**缺點：**

- 治標不治本
- 容易遺漏
- 仍然阻塞執行緒（因為 `.Result`）

---

## 重要觀念整理

### ConfigureAwait(false) 的作用

- ✓ 避免回到原始同步上下文
- ✓ 在執行緒池執行緒繼續執行
- ✗ **不會**影響後續的 await
- ✗ **不會**讓整個方法都脫離上下文

### 黃金規則

1. **永遠不要在 ASP.NET 中混用 `.Result` 或 `.Wait()` 與 async/await**
2. **整個呼叫鏈都使用 async/await**（async all the way）
3. **在 library 程式碼中使用 `ConfigureAwait(false)`**
4. **在 ASP.NET 應用層不需要 `ConfigureAwait(false)`**

---

## 實務建議（針對你的專案）

### GTS 系統 (.NET Framework 4.8)

- 特別注意死鎖問題
- 必須全部 async/await 或全部 ConfigureAwait(false)
- 建議：Controller 方法改用 `async Task<ActionResult>`

### Portal 系統 (.NET Core 3.1)

- 較不容易死鎖
- 但仍應遵循 async/await 最佳實踐
- 避免使用 `.Result` 和 `.Wait()`
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
    // ▼ 階段 1：在原始 ASP.NET request context (請求執行緒 A)
    // ========================================
    // 👇 重要！狀態機在「第一個 await」時就捕捉了 SynchronizationContext (SyncCtx-A)
    //    並將它儲存在狀態機的「背包」中，之後會一直保存著
    //    就像放了一個 GPS 定位器，指向「請求執行緒 A」
    await Task.Delay(1000).ConfigureAwait(false);
    // 👆 ConfigureAwait(false) 只影響「這一個」await：
    //    告訴狀態機：「這次等待結束後，不用回到 SyncCtx-A」
    //    但注意！狀態機仍然保存著 SyncCtx-A，並沒有清除它
    
    // ▼ 階段 2：已脫離原始上下文，在執行緒池執行緒 (執行緒池執行緒 B) 上執行
    // ========================================
    // 執行緒 (Thread)：實際執行程式碼的工人
    // 同步上下文 (SynchronizationContext)：調度員，決定續行 (Continuation) 要派送到哪裡
    // 
    // 雖然程式碼現在在「執行緒池執行緒 B」上執行，
    // 但狀態機的背包裡仍然帶著 SyncCtx-A 的 GPS 定位器！
    
    await Task.Delay(500);  // 預設 ConfigureAwait(true)
    // 👆 死鎖的關鍵引爆點！
    //    因為沒有 ConfigureAwait(false)，這個 await 會使用預設的 ConfigureAwait(true)
    
    // ▼ 階段 3：嘗試回到原始同步上下文 → 死鎖發生！
    // ========================================
    // 當 500ms 延遲結束後，調度員會：
    // 1. 從狀態機的「背包」裡拿出之前捕捉並儲存的 SyncCtx-A (GPS 定位器)
    // 2. 嘗試把後續程式碼 (return new Data();) 的續行派送回 SyncCtx-A 執行
    // 3. 但發現 SyncCtx-A 正被「請求執行緒 A」的 .Result 呼叫佔用且阻塞中！
    // 
    // 形成死鎖：
    // - 「請求執行緒 A」在等待 Task 完成 (.Result 阻塞)
    // - Task 的續行程式碼在等待「請求執行緒 A」釋放 SyncCtx-A
    // → 雙方陷入無限等待循環！
    // 
    // 🔑 關鍵觀念：
    // - SynchronizationContext 的捕捉與「當前在哪個執行緒」無關
    // - 狀態機從一開始就記住了它應該回去的地方 (SyncCtx-A)
    // - 無論中途在哪個執行緒上執行，只要遇到沒有 ConfigureAwait(false) 的 await
    //   就會嘗試回到最初捕捉到的那個上下文
    // 
    // 💡 函式庫開發黃金法則：
    // 「在函式庫的程式碼中，所有 await 都應該加上 .ConfigureAwait(false)」
    // 因為你永遠不知道呼叫者會不會用 .Result 來阻塞呼叫
    
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
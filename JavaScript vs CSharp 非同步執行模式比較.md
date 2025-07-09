---
date: 2025-07-09 09:52
aliases: 
tags:
  - JavaScript
  - CSharp
  - AJAX
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
#### 📑 [[歡迎]]
#### 📑 [[歡迎]]

---
### JavaScript：通知模式

- **分支執行緒完成** → **通知主執行緒** → **主執行緒接手執行**
- 所有 JavaScript 程式碼都在主執行緒執行 (不包括分支執行緒)
- 非同步操作在其他執行緒，完成後透過 Event Loop 通知主執行緒

### C#：直接執行模式

- **分支執行緒完成** → **直接在該執行緒繼續執行**
- 程式碼可能在不同執行緒執行
- 利用執行緒池，避免執行緒切換成本

---

## JavaScript 執行機制

### 架構組成

- **主執行緒**：執行所有 JavaScript 程式碼
- **Web APIs 執行緒**：處理網路請求、計時器、I/O 操作
- **Event Loop**：管理非同步任務的調度
- **Event Queue**：存放待執行的回調函式

### 執行流程

```javascript
async function example() {
    console.log('1. 主執行緒開始');
    
    // 網路請求交給分支執行緒處理
    const result = await fetch('/api/data');
    
    console.log('3. 回到主執行緒繼續');
}

example();
console.log('2. 主執行緒繼續其他任務');
```

**執行順序：**

1. 主執行緒執行 `console.log('1. 主執行緒開始')`
2. 分支執行緒開始處理 `fetch`，主執行緒暫停該函式
3. 主執行緒繼續執行 `console.log('2. 主執行緒繼續其他任務')`
4. 分支執行緒完成後，通知主執行緒
5. 主執行緒空閒時，執行 `console.log('3. 回到主執行緒繼續')`

### 關鍵特性

- **單一執行緒執行 JavaScript 程式碼**
- **Event Queue 排隊等待**
- **主執行緒忙碌時，回調函式需等待**

---

## C# 執行機制

### 架構組成

- **主執行緒**：啟動非同步操作
- **執行緒池**：處理非同步任務
- **TaskScheduler**：管理任務調度
- **SynchronizationContext**：控制執行緒切換

### 執行流程

```csharp
async Task Example()
{
    Console.WriteLine($"1. 開始 - 執行緒 {Thread.CurrentThread.ManagedThreadId}");
    
    await SomeAsyncOperation();
    
    Console.WriteLine($"2. 繼續 - 執行緒 {Thread.CurrentThread.ManagedThreadId}");
    // 可能在不同執行緒執行
}
```

**執行順序：**

1. 主執行緒開始執行
2. 執行緒池中的執行緒處理非同步操作
3. 操作完成後，**可能直接在該執行緒繼續執行**
4. 不需要切換回原始執行緒

### 關鍵特性

- **多執行緒執行程式碼**
- **效能優化，避免執行緒切換**
- **可能在不同執行緒繼續執行**

---

## 實際範例比較

### 主執行緒忙碌情況

**JavaScript：**

```javascript
async function asyncTask() {
    console.log('非同步任務開始');
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1秒完成
    console.log('非同步任務結束'); // 需等待主執行緒空閒
}

asyncTask();

// 主執行緒忙碌 3 秒
console.log('主執行緒開始忙碌');
for (let i = 0; i < 1000000000; i++) {
    // 忙碌中...
}
console.log('主執行緒忙碌結束');

// 輸出：非同步任務開始 → 主執行緒開始忙碌 → 主執行緒忙碌結束 → 非同步任務結束
```

**C#：**

```csharp
async Task AsyncTask()
{
    Console.WriteLine("非同步任務開始");
    await Task.Delay(1000); // 1秒完成
    Console.WriteLine("非同步任務結束"); // 可能立即在執行緒池執行緒執行
}

AsyncTask();

// 主執行緒忙碌 3 秒
Console.WriteLine("主執行緒開始忙碌");
Thread.Sleep(3000);
Console.WriteLine("主執行緒忙碌結束");

// 輸出：非同步任務開始 → 主執行緒開始忙碌 → 非同步任務結束 → 主執行緒忙碌結束
```

---

## 設計原因分析

### JavaScript 選擇通知模式的原因

1. **DOM 安全性**：避免多執行緒同時操作 DOM
2. **簡化程式設計**：不需要考慮執行緒同步問題
3. **UI 響應性**：確保 UI 更新在正確執行緒執行

### C# 選擇直接執行模式的原因

1. **效能優化**：減少執行緒切換成本
2. **資源利用**：充分利用執行緒池
3. **靈活性**：可透過 SynchronizationContext 控制行為

---

## 實務影響

### JavaScript 開發注意事項

- 避免在主執行緒進行長時間同步操作
- 使用 Web Workers 處理密集計算
- 理解 Event Loop 的運作機制

### C# 開發注意事項

- UI 更新需要切換到 UI 執行緒
- 使用 `ConfigureAwait(false)` 避免不必要的執行緒切換
- 了解 SynchronizationContext 的作用

---

## 總結

|特性|JavaScript|C#|
|---|---|---|
|執行模式|通知模式|直接執行模式|
|執行緒使用|單執行緒執行程式碼|多執行緒執行程式碼|
|完成後行為|通知主執行緒接手|直接在當前執行緒繼續|
|主要優勢|執行緒安全、簡化程式設計|效能優化、資源利用|
|主要挑戰|主執行緒阻塞影響回調|執行緒切換複雜性|
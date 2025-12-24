---
date: 2025-08-14 12:13
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
## 一、核心概念

這兩個詞描述了系統如何處理多個任務,但角度完全不同。

### 並行 (Concurrency)
- **定義**: 一種**處理**多個任務的能力。它是程式的**結構設計**,重點在於任務的切分與調度。
- **運作方式**: 任務在時間上交錯執行 (Interleaved)。系統在單一時間點只做一件事,但在任務之間快速切換,尤其是在等待的空檔。
- **目標**: 避免資源浪費在「等待」上,提升系統的**回應速度**與資源利用率。

### 平行 (Parallelism)
- **定義**: 一種**同時執行**多個任務的能力。它是硬體的**執行方式**,重點在於任務在物理上的同時發生。
- **運作方式**: 需要多個處理單元(如多核心 CPU),讓多個任務真正地在同一時刻被執行。
- **目標**: 縮短完成所有任務的**總時間**,提升系統的**吞吐量 (Throughput)**。

---

## 二、經典比喻:廚師與晚宴

這個比喻最能幫助我們直觀地理解差異。

### 並行 (Concurrency) → 一位廚師,多道菜
- 一位厲害的廚師要獨自準備湯、牛排、沙拉。
- 他會先開火燉湯(等待),然後利用空檔去烤牛排(等待),再利用空檔去切沙拉(執行)。
- **精髓**: 廚師(執行緒)只有一位,但他不會被「等待」卡住,而是在不同任務間智慧切換,看起來像同時在忙所有事。

### 平行 (Parallelism) → 一個廚師團隊,分工合作
- 三位廚師分別負責湯、牛排、沙拉。
- 三個人在**同一時間**,各自專心做自己的菜。
- **精髓**: 有多位廚師(多個 CPU 核心),讓多項任務真正地同時被執行。

---

## 三、在 .NET 中的應用場景與模式

### 並行 (Concurrency) 的應用:`async / await`

**適用場景**: **I/O 密集型 (I/O-Bound)** 工作
- 網路請求 (`HttpClient`)
- 檔案讀寫 (`FileStream`)
- 資料庫操作 (`Entity Framework Async`)

**目的**: 當前執行緒在發起一個需要「等待」的 I/O 操作後,可以被釋放出來去做其他事(例如:保持 UI 回應、處理其他請求),而不是被阻塞。

**範例程式碼**:
```csharp
// 適合「並行」的場景:非同步處理多個檔案
async Task ProcessAllFilesAsync(IEnumerable<FileInfo> files)
{
    // 立即啟動所有檔案的讀取任務,並收集這些 "進行中" 的 Task
    var processingTasks = files.Select(file => ProcessSingleFileAsync(file));
    
    // 非同步地等待所有任務完成
    await Task.WhenAll(processingTasks);
}

async Task ProcessSingleFileAsync(FileInfo file)
{
    // 遇到 await,執行緒會被釋放,直到 I/O 操作完成
    var content = await File.ReadAllTextAsync(file.FullName);
    // ... 接續處理
}
```

### 平行 (Parallelism) 的應用:`Task.Run` 與 `Parallel.ForEach`

**適用場景**: **CPU 密集型 (CPU-Bound)** 工作
- 複雜的數學運算
- 圖片或影片處理
- 大量的資料集合計算

**目的**: 將耗時的計算工作分配到多個 CPU 核心上,利用硬體能力來加速完成。

**範例程式碼**:
```csharp
// 適合「平行」的場景:對大量資料進行耗時的 CPU 運算
void IdentifyAllParts(IEnumerable<Part> parts)
{
    // Task.Run 將每個運算任務丟到背景執行緒集區 (ThreadPool)
    var identificationTasks = parts.Select(part => Task.Run(() =>
    {
        var identifier = new PartTypeIdentifier(part);
        identifier.ResolvePartType(); // 這是耗費 CPU 的工作
    }));

    // 等待所有平行的運算任務結束
    Task.WaitAll(identificationTasks.ToArray());
}
```

---

## 四、快速對照表

| 特性 | 並行 (Concurrency) | 平行 (Parallelism) |
|:---|:---|:---|
| **核心概念** | 結構上的**調度** | 硬體上的**同時執行** |
| **比喻** | 一位廚師,多道菜 | 多位廚師,各做一道菜 |
| **目標** | 避免等待,提升**回應速度** | 縮短總耗時,提升**吞吐量** |
| **執行方式** | 交錯執行 (Interleaved) | 同時執行 (Simultaneous) |
| **硬體需求** | 單核心即可 | **必須**多核心 |
| **.NET 模式** | `async / await` | `Task.Run`, `Parallel.ForEach` |

---

## 五、結論與關鍵

- **並行是一種設計,平行是一種執行。**
- 你可以寫出**並行**的程式碼,讓它在單核心電腦上交錯執行;如果這台電腦剛好是多核心的,那麼這些並行的任務就**可能**會被平行地執行。
- **先有好的並行設計,才能有效地利用硬體達成平行執行**。
---
title: Task.CompletedTask、Task.FromResult
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5]

---


Task.CompletedTask 是一個預先完成的 Task 物件，代表一個已經完成且沒有回傳值的工作。它在 .NET Framework 4.6 和 .NET Core 中被引入，主要用於需要回傳 Task，但實際上不需要進行非同步操作的場景。

以下是一個實際的例子:

```csharp
public async Task ProcessDataAsync(bool shouldProcess)
{
    if (!shouldProcess)
    {
        // 不需要處理時直接返回完成的 Task
        return Task.CompletedTask;
    }
    
    // 進行實際的非同步處理
    await DoSomeWorkAsync();
}
```

Task.FromResult 則是用來建立一個已完成且帶有特定回傳值的 Task。這個方法在我們需要立即回傳一個有值的 Task 時特別有用。

讓我們看一個具體例子:

```csharp
public async Task<int> GetValueAsync(bool useCache)
{
    if (useCache && _cachedValue != null)
    {
        // 如果有快取值,直接返回包裝成 Task 的結果
        return Task.FromResult(_cachedValue.Value);
    }
    
    // 否則進行實際的非同步運算
    return await CalculateValueAsync();
}
```

這兩個方法的主要差異在於:
1. Task.CompletedTask 回傳的是 Task 類型(無回傳值)
2. Task.FromResult<T> 回傳的是 Task<T> 類型(有泛型回傳值)

使用這些方法的好處是:
- 提高效能 - 不需建立新的 Task 物件
- 程式碼更簡潔 - 不需要使用 async/await
- 符合介面設計 - 可以保持方法簽章的一致性
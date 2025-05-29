---
title: async / await 範例
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5, .Entity Framework Core, .Entity Framework]

---

```csharp=
[HttpGet]
public async Task<CommonResponse<SpmCostKpDataVM>> FindSpmCostKpData([FromQuery] ModifyByIdVM model)
{
    var categoryTask = GetCategoryAsync(model.ModifyId);    // 取得 Category
    var columnsTask = GetColumnsAsync(model.ModifyId);  // 取得 Columns

    // 同時執行兩個任務，並且等待任務完成
    await Task.WhenAll(categoryTask, columnsTask);
    vmKpData.KpCategory = await categoryTask;
    vmKpData.KpColumns = await columnsTask;
}

private async Task<SpmCostKpCatVM> GetCategoryAsync(int categoryId)
{
    using var scope = _serviceProvider.CreateScope();
    var _newDb = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();

    var condition = ExpressionBuilder.True<SpmcostKpcategory>();
    condition = condition.And(dbKpCate => dbKpCate.IsValid.Value && dbKpCate.Id == categoryId);

    return await _newDb.SpmcostKpcategory.AsNoTracking()
        .Where(condition)
        .Select(dbKpCate => new SpmCostKpCatVM
        {
            Id = dbKpCate.Id,
            Name = dbKpCate.Name,
            MktSpecId = dbKpCate.MktspecId,
            MktSpecName = (dbKpCate.Mktspec != null && dbKpCate.Mktspec.IsValid.Value) ? dbKpCate.Mktspec.Spec : null
        }).FirstOrDefaultAsync();
}

private async Task<List<SpmCostKpColVM>> GetColumnsAsync(int categoryId)
{
    using var scope = _serviceProvider.CreateScope();
    var _newDb = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();

    return await _newDb.SpmcostKpcolumn.AsNoTracking()
        .Where(dbKpCol => dbKpCol.IsValid.Value && dbKpCol.KpcategoryId == categoryId)
        .OrderBy(dbKpCol => dbKpCol.Sequence)
        .Select(dbKpCol => new SpmCostKpColVM
        {
            Id = dbKpCol.Id,
            Name = dbKpCol.Name,
            IsKeypartList = dbKpCol.IsKeypartList
        }).ToListAsync();
}
```

# 問題一
```!
我記得 await _newDb.SpmcostKpcategory.FirstOrDefaultAsync(); 中，如果有加上 await 的話，程序還是會等這行程式碼執行完後，才會執行下一行，但這樣的話，範例還是會同時執行 categoryTask、columnsTask 嗎？

為什麼 model.ModifyId = await GetCategoryIdByKeyPartIdAsync(model.ModifyId_1); 要加上 await，而 var categoryTask = GetCategoryAsync(model.ModifyId); 卻不用加上 await？
```

讓我詳細解釋這兩個關於非同步程式設計的重要問題。

(1) 關於並行執行的機制：

你說得對，當我們使用 await 時，程式確實會等待該行完成才繼續執行。但是關鍵在於「這段等待時間內發生了什麼」。讓我用一個餐廳的例子來解釋：

```csharp
// 這種寫法會依序執行（就像一個服務生依序處理兩個訂單）
var result1 = await GetCategoryAsync(id);        // 等待第一個查詢完成
var result2 = await GetColumnsAsync(id);         // 然後才開始第二個查詢

// 這種寫法會並行執行（就像同時派出兩個服務生處理訂單）
var categoryTask = GetCategoryAsync(id);         // 立即開始第一個查詢
var columnsTask = GetColumnsAsync(id);           // 立即開始第二個查詢
await Task.WhenAll(categoryTask, columnsTask);   // 等待兩個查詢都完成
```

在第二種寫法中，即使 GetCategoryAsync 和 GetColumnsAsync 內部都使用了 await，這兩個查詢仍然會並行執行，因為：
1. 當我們呼叫 GetCategoryAsync 時，它會立即開始查詢
2. 不等第一個查詢完成，就立即開始呼叫 GetColumnsAsync
3. 兩個查詢同時在進行中
4. Task.WhenAll 會等待兩個查詢都完成

就像餐廳裡，一個服務生在等待廚房做第一道菜時，另一個服務生可以同時處理第二份訂單。

(2) 關於不同情境使用 await 的原因：

```csharp
// 這裡需要 await 因為我們需要立即使用查詢結果
model.ModifyId = await GetCategoryIdByKeyPartIdAsync(model.ModifyId_1);
// 我們需要知道 ModifyId 的值才能繼續後面的操作

// 這裡不需要立即 await 因為我們暫時不需要查詢結果
var categoryTask = GetCategoryAsync(model.ModifyId);
var columnsTask = GetColumnsAsync(model.ModifyId);
// 我們可以讓這些查詢在背景執行

// 直到這裡我們才真正需要結果
await Task.WhenAll(categoryTask, columnsTask);
```

這就像是：
1. 第一種情況：你點餐時需要先確認餐廳有沒有這道菜（必須等待確認）
2. 第二種情況：你已經確認有這道菜，就可以直接下單，不用立即等待（可以稍後再來取餐）

所以使用 await 的時機取決於：
- 我們是否需要立即使用查詢結果
- 是否有機會讓多個操作並行執行

在第一個例子中，我們需要 ModifyId 的值來執行後續的查詢，所以必須等待。而在第二個例子中，我們可以讓多個查詢同時進行，最後再一起取得結果，這樣可以節省總體執行時間。

# 問題二
```!
依您所說，請問「GetCategoryAsync 和 GetColumnsAsync 內部都使用了 await」這個部份，是不是也可以不用寫 await？
```

讓我深入解釋這個概念。這是一個很好的問題，因為它觸及了非同步程式設計的核心概念。

首先，讓我們看看如果不使用 await 會發生什麼：

```csharp
private Task<SpmCostKpCatVM> GetCategoryAsync(int categoryId)  // 注意返回型別是 Task<T>
{
    using var scope = _serviceProvider.CreateScope();
    var db = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();

    return db.SpmcostKpcategory.AsNoTracking()  // 直接返回 Task
        .Where(condition)
        .Select(dbKpCate => new SpmCostKpCatVM { ... })
        .FirstOrDefaultAsync();
}
```

這個寫法有一個嚴重的問題：`using` 區塊會在方法立即結束時關閉，而不是等到資料庫查詢完成。這可能導致：
1. 資料庫連線在查詢完成前就被關閉
2. DbContext 在查詢完成前就被釋放
3. 最終造成查詢失敗

讓我用一個生活中的例子來解釋：
想像你去餐廳點餐。如果服務生在你點完餐後立即下班回家（相當於立即關閉 using 區塊），而不是等到你的餐點送到桌上，這顯然會造成問題。

所以正確的寫法應該是：

```csharp
private async Task<SpmCostKpCatVM> GetCategoryAsync(int categoryId)
{
    using var scope = _serviceProvider.CreateScope();
    var db = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();

    // await 確保資料庫查詢完成後，using 區塊才會結束
    return await db.SpmcostKpcategory.AsNoTracking()
        .Where(condition)
        .Select(dbKpCate => new SpmCostKpCatVM { ... })
        .FirstOrDefaultAsync();
}
```

在這個版本中：
1. await 確保資料庫查詢完成
2. 資源（DbContext 和資料庫連線）會被正確管理
3. 程式碼的行為更可預測

總結來說：
- 當我們需要在方法內部管理資源（如使用 using 語句）時，我們必須使用 await
- 當方法只是簡單地傳遞 Task 時，我們可以不使用 await
- 使用 await 不會影響查詢的並行性，因為並行性是由呼叫方式決定的

這就像餐廳服務生必須等到客人收到餐點後才能結束服務，但廚房可以同時準備多道菜。await 確保了服務的完整性，而並行性則是由工作分配方式決定的。

# 問題三
```!
我本來以為執行 GetCategoryAsync 的第一行程式碼後，馬上又會執行 GetColumnsAsync 的第一行程式碼，但看起來並沒有
```
讓我詳細解釋當我們執行這些非同步方法時實際發生的情況。你的觀察很敏銳，確實這裡的執行流程可能和我們預期的不太一樣。

讓我們仔細看看代碼的執行順序：

```csharp
private async Task<SpmCostKpCatVM> GetCategoryAsync(int categoryId)
{
    // 步驟 1: 創建 scope
    using var scope = _serviceProvider.CreateScope();
    // 步驟 2: 獲取 DbContext
    var db = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();
    // 步驟 3: 執行查詢
    return await db.SpmcostKpcategory.AsNoTracking()...
}

// 主方法中
var categoryTask = GetCategoryAsync(id);
var columnsTask = GetColumnsAsync(id);
```

實際的執行順序是這樣的：
1. 當我們呼叫 `GetCategoryAsync` 時，它會執行到第一個 await 之前的所有同步代碼（包括創建 scope 和獲取 DbContext）
2. ==只有當遇到 await 時，控制權才會返回到主方法==
3. 然後開始執行 `GetColumnsAsync`

這就像是餐廳的點餐流程：
- 第一個服務生需要先拿到點餐本和筆（創建 scope 和獲取 DbContext）
- 然後才開始寫下訂單（開始查詢）
- 第二個服務生必須等第一個服務生拿到工具後，才能開始他的工作



然而，需要注意的是，即使有這個小小的序列化過程（一個接一個創建資源），整體的效能影響通常是很小的，因為：
1. 創建 scope 和獲取 DbContext 的時間相對於資料庫查詢來說是微不足道的
2. 真正耗時的是資料庫查詢本身，而這部分確實是並行執行的


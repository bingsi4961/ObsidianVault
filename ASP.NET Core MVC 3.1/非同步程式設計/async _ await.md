---
title: async / await
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5, .Entity Framework Core, .Entity Framework]

---

同步處理（Synchronous）：
```csharp
// 同步處理的例子
public SpmCostKpDataVM GetData()
{
    // 主執行緒會在這裡等待，直到資料庫查詢完成
    var result = _db.SpmcostKpcategory.FirstOrDefault();
    // 在資料庫查詢完成之前，不會執行到這一行
    return result;
}
```

非同步處理（Asynchronous）：
```csharp
// 非同步處理的例子
public async Task<SpmCostKpDataVM> GetDataAsync()
{
    // 主執行緒不會被阻塞，而是釋放出來做其他工作
    var result = await _db.SpmcostKpcategory.FirstOrDefaultAsync();
    // 當資料庫查詢完成後，才會繼續執行這一行
    return result;
}
```

現在，讓我們逐步解析之前提供的範例：

```csharp
// 1. 方法宣告
public async Task<SpmCostKpDataVM> GetCategoryDataAsync(int modifyId)
{
    // 第一個查詢不使用 async，因為我們需要立即確認是否有數據
    var categoryQuery = _db.SpmcostKpcategory
        .AsNoTracking()
        .Where(x => x.IsValid.Value && x.Id == modifyId)
        .Select(x => new SpmCostKpDataVM
        {
            KpCategory = new SpmCostKpCatVM
            {
                Id = x.Id,
                Name = x.Name,
                MktSpecId = x.MktspecId,
                MktSpecName = (x.Mktspec != null && x.Mktspec.IsValid.Value) 
                    ? x.Mktspec.Spec 
                    : null
            }
        })
        .FirstOrDefault();

    // 如果找不到資料，立即返回
    if (categoryQuery == null)
        return null;

    // 2. 使用 Task.WhenAll 同時執行多個非同步操作
    // 這樣可以同時查詢 Columns、Classes，而不用等待彼此完成
    var columnsTask = _db.SpmcostKpcolumn
        .AsNoTracking()
        .Where(x => x.IsValid.Value && x.KpcategoryId == modifyId)
        .OrderBy(x => x.Sequence)
        .Select(x => new SpmCostKpColVM
        {
            Id = x.Id,
            Name = x.Name,
            IsKeypartList = x.IsKeypartList
        })
        .ToListAsync();  // 注意這裡用 ToListAsync()

    var classesTask = _db.SpmcostClass
        .AsNoTracking()
        .Where(x => x.IsValid.Value && x.KpcategoryId == modifyId)
        .OrderBy(x => x.Sequence)
        .Select(x => new SpmCostClassVM
        {
            Id = x.Id,
            Name = x.Name
        })
        .ToListAsync();  // 注意這裡用 ToListAsync()

    // 等待兩個查詢都完成
    await Task.WhenAll(columnsTask, classesTask);

    // 將結果賦值給 categoryQuery
    categoryQuery.KpColumns = await columnsTask;
    categoryQuery.Classes = await classesTask;

    // 3. 處理 KeyParts 查詢
    var keyPartColumnIds = categoryQuery.KpColumns
        .Where(x => x.IsKeypartList)
        .Select(x => x.Id)
        .ToList();

    categoryQuery.KeyParts = await _db.SpmcostKeypart
        .AsNoTracking()
        .Where(x => x.IsValid.Value && keyPartColumnIds.Contains(x.KpcolumnId))
        .Select(x => new SpmCostKeyPartVM
        {
            Id = x.Id,
            KPColumnId = x.KpcolumnId,
            MKTSpecInfoId = x.MktspecInfoId,
            KpName = x.Name,
            Cost = x.Cost
        })
        .ToListAsync();  // 注意這裡用 ToListAsync()

    // 4. 處理 Descriptions 查詢
    var descColumnIds = categoryQuery.KpColumns
        .Where(x => !x.IsKeypartList)
        .Select(x => x.Id)
        .ToList();

    categoryQuery.KpDescriptions = await _db.SpmcostKeypartDescription
        .AsNoTracking()
        .Where(x => descColumnIds.Contains(x.KpcolumnId))
        .Select(x => new SpmCostKpDescriptionVM
        {
            KeyPartId = x.KeypartId,
            KPColumnId = x.KpcolumnId,
            Description = x.Description
        })
        .ToListAsync();  // 注意這裡用 ToListAsync()

    return categoryQuery;
}
```

關鍵概念解釋：

1. async/await 關鍵字：
   - `async`: 標記這個方法是非同步的
   - `await`: 等待一個非同步操作完成，但不會阻塞主執行緒

2. Task<T>：
   - 代表一個非同步操作，最終會返回型別 T 的結果
   - 例如：`Task<List<SpmCostKpColVM>>` 表示這個非同步操作完成時會返回一個 `List<SpmCostKpColVM>`

3. Task.WhenAll：
   - 同時執行多個非同步操作
   - 等待所有操作都完成才繼續執行
   - 可以提高效能，因為多個查詢可以並行執行

4. ToListAsync()：
   - Entity Framework Core 提供的非同步方法
   - 替代同步的 ToList()，可以避免阻塞資料庫連接

使用非同步的好處：

1. 提升效能：
   - 可以同時執行多個資料庫查詢
   - 在等待資料庫回應時，不會阻塞應用程式的其他操作

2. 更好的使用者體驗：
   - 應用程式在等待資料庫操作時仍然可以響應使用者操作
   - 避免界面凍結

要使用這個非同步方法，你需要這樣呼叫它：

```csharp
// 在控制器中
public async Task<IActionResult> GetData(int id)
{
    var result = await _service.GetCategoryDataAsync(id);
    return Json(result);
}
```

# 問題一
```
1. 在 ToListAsync 的最前面，是不是要加 await？
2. 如果不加 await Task.WhenAll 會變成什麼樣？
3. 呼叫非同步方法，我可以不加 await/async 嗎？ 不然的話，我所有的 function 都要加上 async Task，很麻煩
```

(1) 關於 ToListAsync() 前面是否需要加 await：

這取決於你的使用情境。讓我用例子說明：

```csharp
// 情況 1：直接使用 ToListAsync()
var columnsTask = _db.SpmcostKpcolumn
    .Where(x => x.IsValid)
    .ToListAsync();
// columnsTask 的型別是 Task<List<SpmcostKpcolumn>>

// 情況 2：加上 await
var columns = await _db.SpmcostKpcolumn
    .Where(x => x.IsValid)
    .ToListAsync();
// columns 的型別是 List<SpmcostKpcolumn>
```

在我之前提供的範例中，因為我們要使用 Task.WhenAll 同時執行多個查詢，所以故意不加 await，保持它是 Task 型別。這樣可以讓多個查詢並行執行，提升效能。

(2) 如果不使用 Task.WhenAll：

讓我用例子來說明差異：

```csharp
// 使用 Task.WhenAll 的情況
var columnsTask = _db.SpmcostKpcolumn.ToListAsync();
var classesTask = _db.SpmcostClass.ToListAsync();
await Task.WhenAll(columnsTask, classesTask);

// 不使用 Task.WhenAll 的情況
var columns = await _db.SpmcostKpcolumn.ToListAsync();
var classes = await _db.SpmcostClass.ToListAsync();
```

主要差異是：
- 使用 Task.WhenAll：兩個查詢會同時執行，總耗時約等於較慢的那個查詢
- 不使用 Task.WhenAll：查詢會依序執行，總耗時約等於兩個查詢時間的總和

(3) 關於是否每個方法都要加上 async/await：

你可以用幾種方式來處理這個情況：

1. 同步包裝非同步方法（不建議）：
```csharp
// 不建議的方式
public SpmCostKpDataVM GetData(int id)
{
    // 這會造成死鎖風險
    return GetDataAsync(id).Result;
}
```

2. 使用 Task.Run 執行同步方法（某些情況可用）：
```csharp
public void ProcessData(int id)
{
    // 在背景執行非同步操作
    Task.Run(async () => {
        var result = await GetDataAsync(id);
        // 處理結果
    });
}
```

3. 依據應用場景選擇性使用 async/await：
```csharp
// API 控制器 - 建議使用 async
public class ApiController : ControllerBase 
{
    public async Task<IActionResult> GetData(int id)
    {
        var result = await _service.GetDataAsync(id);
        return Ok(result);
    }
}

// 背景工作 - 可以使用同步方法
public class BackgroundJob
{
    public void ProcessData(int id)
    {
        // 同步處理
        var result = GetSyncData(id);
        // 處理結果
    }
}
```

# 問題二
```!
像你說的「不使用 Task.WhenAll：查詢會依序執行」，那像 var columns = await _db.SpmcostKpcolumn.ToListAsync(); 這一行，還加上 await/ ToListAsnync，好像就沒有意義了，反正都是要依序執行
```
首先，我們要理解非同步程式設計有兩個主要目標：
1. 並行處理（Parallelism）：同時執行多個任務
2. 非阻塞操作（Non-blocking）：在等待長時間操作時釋放執行緒

讓我用一個生活中的例子來解釋：

想像你在餐廳點餐。有兩種服務方式：

同步方式（阻塞）：
```csharp
// 同步寫法
public List<SpmcostKpcolumn> GetData()
{
    var result = _db.SpmcostKpcolumn.ToList(); // 服務生站在你桌前等待，直到廚房完成料理
    return result;
}
```
這就像服務生接到你的訂單後，站在你的桌子前等待，直到廚房把菜做好才去服務其他客人。

非同步方式（非阻塞）：
```csharp
// 非同步寫法
public async Task<List<SpmcostKpcolumn>> GetData()
{
    var result = await _db.SpmcostKpcolumn.ToListAsync(); // 服務生可以去服務其他客人
    return result;
}
```
這就像服務生接到你的訂單後，先去服務其他客人，等廚房通知菜做好了才回來上菜 (return result)。

即使你只點一道菜（單一查詢），非同步操作仍然是有價值的，因為：

1. 執行緒效率：
```csharp
// 在 Web 應用程式中
public async Task<IActionResult> GetData()
{
    // 等待資料庫時，IIS執行緒可以去處理其他請求
    var data = await _db.SpmcostKpcolumn.ToListAsync();
    return Json(data);
}
```

2. 使用者介面響應性：
```csharp
// 在 Windows Forms 或 WPF 應用程式中
private async void button1_Click(object sender, EventArgs e)
{
    // 等待資料庫時，UI執行緒不會被凍結
    var data = await _db.SpmcostKpcolumn.ToListAsync();
    dataGridView1.DataSource = data;
}
```

所以，即使查詢是依序執行的，使用 await/ToListAsync 仍然很重要，因為：
1. 它讓執行緒在等待資料庫回應時可以去做其他工作
2. 在 Web 應用程式中，可以處理更多的並發請求
3. 在桌面應用程式中，可以保持使用者介面的響應性

補充說明：如果你真的需要依序執行多個查詢，且後面的查詢依賴於前面查詢的結果，那麼使用 await 而不是 Task.WhenAll 是正確的選擇：

```csharp
// 當查詢之間有依賴關係時
var columns = await _db.SpmcostKpcolumn.ToListAsync();
var keyPartColumnIds = columns.Where(x => x.IsKeypartList).Select(x => x.Id).ToList();

// 這個查詢需要使用上一個查詢的結果
var keyParts = await _db.SpmcostKeypart
    .Where(x => keyPartColumnIds.Contains(x.KpcolumnId))
    .ToListAsync();
```

但如果查詢之間沒有依賴關係，使用 Task.WhenAll 讓它們並行執行會更有效率：

```csharp
// 當查詢之間沒有依賴關係時
var columnsTask = _db.SpmcostKpcolumn.ToListAsync();
var classesTask = _db.SpmcostClass.ToListAsync();

// 同時執行兩個查詢
await Task.WhenAll(columnsTask, classesTask);
```

所以，即使在單一查詢的情況下，使用非同步方法仍然是很有價值的，因為它能提升系統的整體效能和響應性。
    
# 問題三
```csharp
我想確認一下我得到的觀念是否正確

=> 第一種情境如下：
var columnsTask = await _db.SpmcostKpcolumn.ToListAsync();    // 這是主執行緒??
var classesTask = await _db.SpmcostClass.ToListAsync();        // 這是主執行緒??
(1) columnsTask、classesTask 會依序執行
(2) columnsTask 的型別是 List<SpmcostKpcolumn>
(3) classesTask 的型別是 List<SpmcostClass>
(4) 但當 columnsTask、classesTask 在執行時，主執行緒會先去做應用程序的別的事情 (但不是執行 classesTask)

=> 第二種情境如下：
var columnsTask = _db.SpmcostKpcolumn.ToListAsync();    // 這是分支執行緒??
var classesTask = _db.SpmcostClass.ToListAsync();        // 這是分支執行緒??
await Task.WhenAll(columnsTask, classesTask);            // 這是主執行緒??
(1) columnsTask、classesTask 可以同時進行查詢
(2) columnsTask 的型別是 Task<List<SpmcostKpcolumn>>
(3) classesTask 的型別是 Task<List<SpmcostClass>>
(4) Task.WhenAll 會等待 columnsTask、classesTask 都執行完成後，才會再執行後面的程式碼
```
    
你的理解大致正確，讓我補充解釋一些細節。

關於第一種情境：
```csharp
var columns = await _db.SpmcostKpcolumn.ToListAsync();    
var classes = await _db.SpmcostClass.ToListAsync();       
```

你的理解幾乎都對，但有一點需要澄清：這裡不太準確說是「主執行緒」在做其他事。讓我解釋得更精確一些：

當程式執行到第一個 await 時，ASP.NET Core（假設是 Web 應用程式）會：
1. 釋放當前的請求處理執行緒回到執行緒池
2. 在資料庫操作完成時，從執行緒池取得一個執行緒（可能是不同的執行緒）繼續執行
3. 這個釋放和重新獲取執行緒的過程，讓伺服器能夠處理其他進來的請求

所以更準確的說法是：在等待資料庫操作時，執行緒被釋放去處理其他請求，而不是「主執行緒去做別的事」。

關於第二種情境：
```csharp
var columnsTask = _db.SpmcostKpcolumn.ToListAsync();    
var classesTask = _db.SpmcostClass.ToListAsync();       
await Task.WhenAll(columnsTask, classesTask);           
```

你的理解基本正確，但同樣需要一些澄清：這裡也不太準確說是「分支執行緒」。讓我解釋實際發生的事：

1. `ToListAsync()` 創建的是一個資料庫操作的 Task（任務），而不是直接創建新的執行緒
2. 這些操作會被送到資料庫連接池中執行
3. Entity Framework Core 使用非同步 I/O 操作，這不需要額外的執行緒
4. 當兩個查詢被送到資料庫時，資料庫會同時處理這些請求

讓我用一個類比來解釋：
想像你是一個服務生（執行緒），同時有兩個顧客點餐（兩個查詢）：
- 第一種情境就像你先去廚房點第一個顧客的餐，等做好了再去點第二個
- 第二種情境就像你一次把兩個顧客的點餐單都送到廚房，廚房（資料庫）可以同時準備這兩份餐點

實際的執行過程是：
```csharp
// 第一種情境
var columns = await _db.SpmcostKpcolumn.ToListAsync();
// 執行緒被釋放，等待第一個查詢完成
// 查詢完成後，從執行緒池獲取執行緒繼續執行
var classes = await _db.SpmcostClass.ToListAsync();
// 執行緒再次被釋放，等待第二個查詢完成

// 第二種情境
var columnsTask = _db.SpmcostKpcolumn.ToListAsync();
var classesTask = _db.SpmcostClass.ToListAsync();
// 兩個查詢同時送到資料庫執行
await Task.WhenAll(columnsTask, classesTask);
// 執行緒被釋放，等待兩個查詢都完成
// 當兩個查詢都完成時，從執行緒池獲取執行緒繼續執行
```

這樣的設計讓應用程式能夠：
1. 更有效地使用系統資源（執行緒可以被重複利用）
2. 提高並發處理能力（可以同時處理更多請求）
3. 在等待 I/O 操作時不會浪費執行緒資源
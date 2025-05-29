---
title: async / await 範例 2
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5, .Entity Framework Core, .Entity Framework]

---

```csharp=
[HttpPost]
public async Task<CommonResponse> UpdateSpmCostKpData(SpmCostKpDataVM4Update model)
{
    var resp = new CommonResponse();
    List<TaskContext> taskContexts = null;
    Task[] executedTasks = null;

    try
    {
        taskContexts = await CreateTaskContextsAsync(3);
        
        // 即使某個 task 發生錯誤，其他 task 仍會繼續執行
        // 所有錯誤會被儲存在各自 task 的 Exception 屬性中
        // 等所有 task 執行完畢後，才會進入 catch 區塊
        executedTasks = new[] {
            model.KeyParts.Any(x => x.Id > 0) ? UpdateKeyPartsAndDescsAsync(model, taskContexts[0]) : Task.CompletedTask,
            model.KeyParts.Any(x => x.Id == 0) ? InsertKeyPartsAndDescsAsync(model, taskContexts[1]) : Task.CompletedTask,
            model.DeletedKeyParts?.Any() == true ? DeleteKeyPartsAsync(model, taskContexts[2]) : Task.CompletedTask
        };

        // 等待所有Task都執行結束
        await Task.WhenAll(executedTasks);
        await CommitTransactionsAsync(taskContexts);
        resp.Status = CommonResponse.StatusEnum.Success;
    }
    catch (Exception ex)
    {
        // 這裡的 catch 讓我們有機會等待其他 task 完成
        // 並收集所有 task 的錯誤，而不是只拿到第一個錯誤
        // 當 task 失敗時，例外會被包裝在 Task 物件的 Exception 屬性中c
        // task.Exception 是 AggregateException
        // task.Exception.InnerException 才是實際的例外（如 NullReferenceException）
        
        await RollbackTransactionsAsync(taskContexts);
        resp.Message = FormatErrorMessage(executedTasks, ex);
    }
    finally
    {
        await DisposeContextsAsync(taskContexts);
    }

    return resp;
}

/// <summary>
/// 建立資料庫上下文和交易
/// </summary>
/// <param name="count"></param>
/// <returns></returns>
private async Task<List<TaskContext>> CreateTaskContextsAsync(int count)
{
    var contexts = new List<TaskContext>();

    for (int i = 0; i < count; i++)
    {
        var scope = _serviceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();
        var transaction = await context.Database.BeginTransactionAsync();

        contexts.Add(new TaskContext
        {
            Context = context,
            Transaction = transaction,
            Scope = scope
        });
    }

    return contexts;
}

/// <summary>
/// 提交所有交易
/// </summary>
/// <param name="taskContexts"></param>
/// <returns></returns>
private async Task CommitTransactionsAsync(List<TaskContext> taskContexts)
{
    foreach (var context in taskContexts)
    {
        await context.Transaction.CommitAsync();
    }
}

/// <summary>
/// 回滾所有交易
/// </summary>
/// <param name="taskContexts"></param>
/// <returns></returns>
private async Task RollbackTransactionsAsync(List<TaskContext> taskContexts)
{
    if (taskContexts == null) return;
    foreach (var context in taskContexts.Where(context => context?.Transaction != null))
    {
        await context.Transaction.RollbackAsync();
    }
}

/// <summary>
/// 釋放所有 TaskContext 資源
/// </summary>
/// <param name="taskContexts"></param>
/// <returns></returns>
private async Task DisposeContextsAsync(List<TaskContext> taskContexts)
{
    if (taskContexts == null) return;
    foreach (var context in taskContexts.Where(context => context != null))
    {
        if (context.Transaction != null) await context.Transaction.DisposeAsync();
        if (context.Context != null) await context.Context.DisposeAsync();
        if (context.Scope != null) context.Scope.Dispose();
    }
}

private string FormatErrorMessage(Task[] tasks, Exception ex)
{
    if (tasks == null) { return ex.Message; }

    var errorTasks = tasks.Where(task => task.Exception != null).ToList();
    var errorMsgs = errorTasks.Select(task =>
    {
        var errorMsg = task.Exception.InnerException?.Message ?? task.Exception.Message;
        return errorTasks.Count > 1 ? $"({errorTasks.IndexOf(task) + 1}) {errorMsg}" : errorMsg;
    });

    return string.Join(" ", errorMsgs);
}

/// <summary>
/// 編輯 Keyparts、KpDescriptions
/// </summary>
/// <param name="model"></param>
/// <param name="exceptions"></param>
/// <returns></returns>
private async Task UpdateKeyPartsAndDescsAsync(SpmCostKpDataVM model, TaskContext taskContext)
{
    var _db = taskContext.Context;

    // 從 ViewModel 取得要編輯的 KeyPartId
    var keyPartIds4Vm = model.KeyParts.Where(vmKp => vmKp.Id > 0).Select(vmKp => vmKp.Id);

    // 從 ViewModel 取得要編輯的 KeyPart
    var keyParts4Vm = model.KeyParts.Where(vmKp => vmKp.Id > 0).ToList();

    // 從 DB 取得要編輯的 SpmcostKeypart
    var keyParts4Db = await _db.SpmcostKeypart
        .Where(dbKp => keyPartIds4Vm.Contains(dbKp.Id) && dbKp.IsValid.Value)
        .ToDictionaryAsync(dbKp => dbKp.Id);

    // 從 DB 取得要編輯的 SpmcostKeypartDescription
    var descs4Db = await _db.SpmcostKeypartDescription
        .Where(dbDesc => keyPartIds4Vm.Contains(dbDesc.KeypartId))
        .ToListAsync();

    // 編輯 Keyparts、KpDescriptions
    foreach (var vmKp in keyParts4Vm)
    {
        if (!keyParts4Db.TryGetValue(vmKp.Id, out var entyKp))
        {
            throw new Exception("無法讀取欲編輯的 KeyPart，請重新載入頁面。");
        }

        entyKp.MktspecInfoId = vmKp.MKTSpecInfoId;
        entyKp.Name = vmKp.KpName;
        entyKp.Cost = vmKp.Cost;
        entyKp.ModifyUser = vmKp.ModifyUser;
        entyKp.ModifyDate = DateTime.Now;

        // 從 DB 取得要編輯的 SpmcostKeypartDescription By 單筆 KeyPart
        var descs4Db_singleKp = descs4Db
            .Where(dbDesc => dbDesc.KeypartId == vmKp.Id)
            .ToDictionary(dbDesc => dbDesc.KpcolumnId);

        // 從 View Model 取得要編輯的 SpmCostKpDescriptionVM By 單筆 KeyPart
        var descs4Vm_singleKp = model.KpDescriptions
            .Where(vmDesc => vmDesc.KPs2DescIdx == vmKp.KPs2DescIdx);

        var newKpDescs = new List<SpmcostKeypartDescription>();

        foreach (var vmDesc in descs4Vm_singleKp)
        {
            if (descs4Db_singleKp.TryGetValue(vmDesc.KPColumnId, out var entyDesc))
            {
                entyDesc.Description = vmDesc.Description;
            }
            else
            {
                // 如果是新定義的欄位
                newKpDescs.Add(new SpmcostKeypartDescription
                {
                    KeypartId = vmKp.Id,
                    KpcolumnId = vmDesc.KPColumnId,
                    Description = vmDesc.Description
                });
            }
        }

        if (newKpDescs.Any())
        {
            _db.SpmcostKeypartDescription.AddRange(newKpDescs);
        }
    }

    await _db.SaveChangesAsync();
}

/// <summary>
/// 新增 Keyparts、新增 KpDescriptions 
/// </summary>
/// <param name="model"></param>
/// <param name="exceptions"></param>
/// <returns></returns>
private async Task InsertKeyPartsAndDescsAsync(SpmCostKpDataVM model, TaskContext taskContext)
{
    var dao = new SeqDao();
    var _db = taskContext.Context;

    var newKeyParts = model.KeyParts
        .Where(vmKp => vmKp.Id == 0)
        .Select(vmKp =>
        {
            var newKpId = dao.GetSeqNo("SPMCost_Keypart");
            return new SpmcostKeypart
            {
                Id = newKpId,
                KpcolumnId = vmKp.KPColumnId,
                MktspecInfoId = vmKp.MKTSpecInfoId,
                Name = vmKp.KpName,
                Cost = vmKp.Cost,
                ModifyUser = vmKp.ModifyUser,
                Creator = vmKp.Creator,
                SpmcostKeypartDescription = model.KpDescriptions
                    .Where(vmDesc => vmKp.KPs2DescIdx == vmDesc.KPs2DescIdx)
                    .Select(vmDesc => new SpmcostKeypartDescription
                    {
                        KeypartId = newKpId,
                        KpcolumnId = vmDesc.KPColumnId,
                        Description = vmDesc.Description
                    }).ToList()
            };
        }).ToList();

    if (newKeyParts.Any())
    {
        await _db.SpmcostKeypart.AddRangeAsync(newKeyParts);
        await _db.SaveChangesAsync();
    }
}

```
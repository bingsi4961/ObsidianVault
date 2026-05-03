---
date: 2026-05-03 14:56
title:
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
## TaskDbContextService.cs
```csharp
using GamingAPI.Models.DB;
using Microsoft.EntityFrameworkCore.Storage;
using Microsoft.Extensions.DependencyInjection;
using System;
using System.Collections.Concurrent;
using System.Linq;
using System.Threading.Tasks;

namespace GamingAPI.Handler
{
    // 繼承 IAsyncDisposable，讓 DI 容器在 Request 結束時，自動呼叫 DisposeAsync 以回收資源
    public class TaskDbContextService : IAsyncDisposable
    {
        private readonly IServiceScopeFactory _scopeFactory;
        private readonly ConcurrentQueue<TaskContext> _managedContexts = new ConcurrentQueue<TaskContext>();
        private bool _isDisposed;

        private const string NotInitializedMessage =
            "尚無可操作的 DbContext，請先呼叫 GetWriteContextAsync() / GetReadContext() 建立 DbContext";

        public TaskDbContextService(IServiceScopeFactory scopeFactory)
        {
            _scopeFactory = scopeFactory;
        }

        /// <summary>
        /// 依指定數量，批次建立寫入用的 TaskContext，自動加入內部管理清單
        /// </summary>
        /// <param name="withTransaction">是否開啟 Transaction，預設為 true</param>
        public async Task<TaskContext[]> GetWriteContextsAsync(int count, bool withTransaction = true)
        {
            if (count <= 0) throw new ArgumentOutOfRangeException(nameof(count), "數量必須大於 0");

            var tasks = Enumerable.Range(0, count)
                .Select(_ => GetWriteContextAsync(withTransaction));

            return await Task.WhenAll(tasks);
        }

        /// <summary>
        /// 建立一個寫入用的 TaskContext，自動加入內部管理清單
        /// </summary>
        /// <param name="withTransaction">是否開啟 Transaction，預設為 true</param>
        public async Task<TaskContext> GetWriteContextAsync(bool withTransaction = true)
        {
            if (_isDisposed) throw new ObjectDisposedException(nameof(TaskDbContextService));

            var scope = _scopeFactory.CreateScope();
            var dbContext = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();
            var transaction = withTransaction
                ? await dbContext.Database.BeginTransactionAsync()
                : null;

            var taskContext = new TaskContext
            {
                DbContext = dbContext,
                Transaction = transaction,
                Scope = scope
            };

            _managedContexts.Enqueue(taskContext);
            return taskContext;
        }

        /// <summary>
        /// 建立一個查詢用的 TaskContext，自動加入內部管理清單
        /// 不開啟 Transaction，CommitTransactionsAsync() 時會自動略過
        /// </summary>
        public TaskContext GetReadContext()
        {
            if (_isDisposed) throw new ObjectDisposedException(nameof(TaskDbContextService));

            var scope = _scopeFactory.CreateScope();
            var dbContext = scope.ServiceProvider.GetRequiredService<GamingPortalContextCust>();

            var taskContext = new TaskContext
            {
                DbContext = dbContext,
                Transaction = null,
                Scope = scope
            };

            _managedContexts.Enqueue(taskContext);
            return taskContext;
        }

        /// <summary>
        /// 並行提交所有 Transaction
        /// 提交後立即釋放 Transaction，重複呼叫會安全略過已提交的項目
        /// </summary>
        public async Task CommitTransactionsAsync()
        {
            if (_isDisposed) throw new ObjectDisposedException(nameof(TaskDbContextService));

            if (_managedContexts.IsEmpty)
                throw new InvalidOperationException(NotInitializedMessage);

            var tasks = _managedContexts
                .Where(context => context?.Transaction != null)
                .Select(async context =>
                {
                    try
                    {
                        await context.Transaction.CommitAsync();
                    }
                    finally
                    {
                        // 提交後立即釋放並設為 null，確保重複呼叫時被 Where 安全過濾
                        await context.Transaction.DisposeAsync();
                        context.Transaction = null;
                    }
                });

            await Task.WhenAll(tasks);
        }

        /// <summary>
        /// 並行回滾所有 Transaction
        /// 回滾後立即釋放 Transaction，重複呼叫會安全略過已回滾的項目
        /// </summary>
        public async Task RollbackTransactionsAsync()
        {
            if (_isDisposed) throw new ObjectDisposedException(nameof(TaskDbContextService));

            if (_managedContexts.IsEmpty)
                throw new InvalidOperationException(NotInitializedMessage);

            var tasks = _managedContexts
                .Where(context => context?.Transaction != null)
                .Select(async context =>
                {
                    try
                    {
                        await context.Transaction.RollbackAsync();
                    }
                    finally
                    {
                        // 回滾後立即釋放並設為 null，確保重複呼叫時被 Where 安全過濾
                        await context.Transaction.DisposeAsync();
                        context.Transaction = null;
                    }
                });

            await Task.WhenAll(tasks);
        }

        /// <summary>
        /// 重置目前管理的所有 DbContext 狀態
        /// 會自動釋放所有已建立的 DbContext、Transaction、Scope 資源，避免 Memory Leak
        /// </summary>
        public async Task ResetContextsAsync()
        {
            if (_isDisposed) throw new ObjectDisposedException(nameof(TaskDbContextService));

            await DisposeContextsInternalAsync();
            _managedContexts.Clear();
        }

        /// <summary>
        /// DI 容器在 Request 結束時，自動呼叫 DisposeAsync 以回收資源
        /// </summary>
        public async ValueTask DisposeAsync()
        {
            if (_isDisposed) return;

            await DisposeContextsInternalAsync();
            _managedContexts.Clear();
            _isDisposed = true;
        }

        /// <summary>
        /// 內部並行釋放所有 TaskContext 資源
        /// 釋放順序：Transaction → DbContext → Scope
        /// </summary>
        private async Task DisposeContextsInternalAsync()
        {
            if (_managedContexts.IsEmpty) return;

            var disposeTasks = _managedContexts
                .Where(context => context != null)
                .Select(async context =>
                {
                    // 若 Commit / Rollback 已將 Transaction 設為 null，此處會自動略過
                    if (context.Transaction != null) await context.Transaction.DisposeAsync();
                    if (context.DbContext != null) await context.DbContext.DisposeAsync();
                    context.Scope?.Dispose();
                });

            await Task.WhenAll(disposeTasks);
        }
    }

    public class TaskContext
    {
        public GamingPortalContextCust DbContext { get; set; }
        public IDbContextTransaction Transaction { get; set; }
        public IServiceScope Scope { get; set; }
    }
}

```

## SpmCostKpDataController.cs
```csharp
namespace GamingAPI.Controllers
{
    [Route("api/[controller]/[action]")]
    [ApiController]
    public class SpmCostKpDataController : ControllerBase
    {
        private readonly GamingPortalContextCust _db;
        private readonly TaskDbContextService _taskDbContextService;
        private readonly TaskHelper _taskHelper;

        public SpmCostKpDataController(GamingPortalContextCust context, TaskDbContextService taskDbContextService, TaskHelper taskHelper)
        {
            this._db = context;
            this._taskDbContextService = taskDbContextService;
            this._taskHelper = taskHelper;
        }

        [HttpGet]
        public async Task<CommonResponse<SpmCostKpDataVM>> FindSpmCostKpData([FromQuery] ModifyByIdVM model)
        {
            var resp = new CommonResponse<SpmCostKpDataVM>();
            var vmKpData = new SpmCostKpDataVM();

            var categoryTask = GetCategoryAsync(model.ModifyId);
            var columnsTask = GetColumnsAsync(model.ModifyId);

            await Task.WhenAll(categoryTask, columnsTask);
            vmKpData.KpCategory = await categoryTask;
            vmKpData.KpColumns = await columnsTask;

            resp.Status = CommonResponse.StatusEnum.Success;
            resp.Data = vmKpData;
            return resp;
        }

        /// <summary>
        /// 透過 CategoryId 取得 Category
        /// </summary>
        private async Task<SpmCostKpCatVM> GetCategoryAsync(int categoryId)
        {
            var condition = ExpressionBuilder.True<SpmcostKpcategory>();
            condition = condition.And(dbKpCate => dbKpCate.IsValid.Value && dbKpCate.Id == categoryId);

            var _taskDb = this._taskDbContextService.GetReadContext().DbContext;
            return await _taskDb.SpmcostKpcategory.AsNoTracking()
                .Where(condition)
                .Select(dbKpCate => new SpmCostKpCatVM
                {
                    Id = dbKpCate.Id,
                    Name = dbKpCate.Name,
                    MktSpecId = dbKpCate.MktspecId,
                    MktSpecName = (dbKpCate.Mktspec != null && dbKpCate.Mktspec.IsValid.Value) ? dbKpCate.Mktspec.Spec : null
                }).FirstOrDefaultAsync();
        }

        /// <summary>
        /// 透過 CategoryId 取得 Columns (自訂欄位)
        /// </summary>
        private async Task<List<SpmCostKpColVM>> GetColumnsAsync(int categoryId)
        {
            var _taskDb = this._taskDbContextService.GetReadContext().DbContext;
            return await _taskDb.SpmcostKpcolumn.AsNoTracking()
                .Where(dbKpCol => dbKpCol.IsValid.Value && dbKpCol.KpcategoryId == categoryId)
                .OrderBy(dbKpCol => dbKpCol.Sequence)
                .Select(dbKpCol => new SpmCostKpColVM
                {
                    Id = dbKpCol.Id,
                    Name = dbKpCol.Name,
                    IsKeypartList = dbKpCol.IsKeypartList
                }).ToListAsync();
        }        

        [HttpPost]
        public async Task<CommonResponse> UpdateSpmCostKpData(SpmCostKpDataVM4Update model)
        {
            var resp = new CommonResponse();
            Task[] executedTasks = null;

            try
            {
                executedTasks = new[] {
                    model.KeyParts.Any(x => x.Id > 0)
                        ? UpdateKeyPartsAndDescsAsync(model, await this._taskDbContextService.GetWriteContextAsync())
                        : Task.CompletedTask,
                    model.KeyParts.Any(x => x.Id == 0)
                        ? InsertKeyPartsAndDescsAsync(model, await this._taskDbContextService.GetWriteContextAsync())
                        : Task.CompletedTask
                };

                await Task.WhenAll(executedTasks);
                await this._taskDbContextService.CommitTransactionsAsync();
                resp.Status = CommonResponse.StatusEnum.Success;
            }
            catch (Exception ex)
            {
                await this._taskDbContextService.RollbackTransactionsAsync();
                resp.Message = this._taskHelper.FormatErrorMessage(executedTasks, ex);
            }

            return resp;
        }

        /// <summary>
        /// 新增 Keyparts、新增 KpDescriptions 
        /// </summary>
        private async Task InsertKeyPartsAndDescsAsync(SpmCostKpDataVM model, TaskContext taskContext)
        {
            var dao = new SeqDao();
            var _taskDb = taskContext.DbContext;

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
                        SortOrder = vmKp.SortOrder,
                        ModifyUser = model.setModifyUser,
                        Creator = model.setCreator
                    };
                }).ToList();

            if (newKeyParts.Any())
            {
                await _taskDb.SpmcostKeypart.AddRangeAsync(newKeyParts);
                await _taskDb.SaveChangesAsync();
            }
        }
    }
}
```
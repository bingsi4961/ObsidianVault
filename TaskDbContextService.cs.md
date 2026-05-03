---
date: 2026-05-03 14:56
title:
aliases:
  - 別名測試1
  - 別名測試2
tags:
  - 標籤測試1
  - 標籤測試2

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

---
date: 2025-11-12 11:50
aliases:
tags:
  - Entity_Framework
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
## 版本差異對照

| 功能     | EF6 (.NET Framework)     | EF Core (.NET Core/5+) |
| ------ | ------------------------ | ---------------------- |
| 查詢方法   | `SqlQuery<🚨T>`          | `FromSqlRaw<🚨T>`      |
| 執行命令方法 | `ExecuteSqlCommand`      | `ExecuteSqlRaw`        |
| 適用系統   | GTS (.NET Framework 4.8) | Portal (.NET Core 3.1) |

## 方法選擇判斷規則

|操作類型|需要返回什麼|使用方法|
|---|---|---|
|SELECT|資料集|SqlQuery / FromSqlRaw|
|INSERT (要 ID)|新增的 ID|SqlQuery / FromSqlRaw + OUTPUT|
|INSERT (不要 ID)|影響筆數|ExecuteSqlCommand / ExecuteSqlRaw|
|UPDATE|影響筆數|ExecuteSqlCommand / ExecuteSqlRaw|
|DELETE|影響筆數|ExecuteSqlCommand / ExecuteSqlRaw|

**核心概念**：

- 需要**返回資料** → SqlQuery / FromSqlRaw
- 只需要**影響筆數** → ExecuteSqlCommand / ExecuteSqlRaw

## 基本模型定義

```csharp
public class EmployeeVM
{
    public int EmpId { get; set; }
    public string EmpName { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
    public DateTime HireDate { get; set; }
}
```

## 常見操作範例

### 1. 查詢操作 (SELECT)

**返回資料集，使用 SqlQuery / FromSqlRaw**

```csharp
var params = new List<SqlParameter>();
string sqlstr = @"SELECT EmpId, EmpName, Department, Salary, HireDate 
                  FROM emps WHERE 1=1 ";

if (!string.IsNullOrEmpty(department))
{
    sqlstr += " AND Department = @Department";
    params.Add(new SqlParameter("@Department", department));
}

if (minSalary.HasValue)
{
    sqlstr += " AND Salary >= @MinSalary";
    params.Add(new SqlParameter("@MinSalary", minSalary.Value));
}

var db = new ApplicationDbContext();

// EF6 寫法 (GTS)
var emps = db.Database.SqlQuery<EmployeeVM>(sqlstr, params.ToArray()).ToList();

// EF Core 寫法 (Portal)
var emps = db.Database.FromSqlRaw<EmployeeVM>(sqlstr, params.ToArray()).ToList();

// 結果: emps => List<EmployeeVM>
```

### 2. 新增操作 (INSERT)

#### 方法一：需要取得新增的 ID（使用 OUTPUT）

**返回新增的 ID，使用 SqlQuery / FromSqlRaw**

```csharp
string insertSql = @"
    INSERT INTO Employees (EmpName, Department, Salary, HireDate)
    🚨 OUTPUT INSERTED.EmpId
    VALUES (@EmpName, @Department, @Salary, @HireDate)";

var parameters = new List<SqlParameter>
{
    new SqlParameter("@EmpName", employee.EmpName),
    new SqlParameter("@Department", (object)employee.Department ?? 🚨DBNull.Value),
    new SqlParameter("@Salary", employee.Salary),
    new SqlParameter("@HireDate", employee.HireDate)
};

// EF6 寫法 (GTS)
int newId = db.Database.SqlQuery<int>(insertSql, parameters.ToArray())🚨.First();

// EF Core 寫法 (Portal)
int newId = db.Database.FromSqlRaw<int>(insertSql, parameters.ToArray())🚨.First();
```

#### 方法二：不需要取得 ID（最常用）

**只需要影響筆數，使用 ExecuteSqlCommand / ExecuteSqlRaw**

```csharp
string insertSql = @"
    INSERT INTO Employees (EmpName, Department, Salary, HireDate)
    VALUES (@EmpName, @Department, @Salary, @HireDate)";

var parameters = new List<SqlParameter>
{
    new SqlParameter("@EmpName", employee.EmpName),
    new SqlParameter("@Department", (object)employee.Department ?? DBNull.Value),
    new SqlParameter("@Salary", employee.Salary),
    new SqlParameter("@HireDate", employee.HireDate)
};

// EF6 寫法 (GTS)
int rowsAffected = db.Database.ExecuteSqlCommand(insertSql, parameters.ToArray());

// EF Core 寫法 (Portal)
int rowsAffected = db.Database.ExecuteSqlRaw(insertSql, parameters.ToArray());
```

#### 方法三：分兩段執行取得 ID

**先執行 INSERT，再查詢 ID**

```csharp
string insertSql = @"
    INSERT INTO Employees (EmpName, Department, Salary, HireDate)
    VALUES (@EmpName, @Department, @Salary, @HireDate)";

var parameters = new List<SqlParameter>
{
    new SqlParameter("@EmpName", employee.EmpName),
    new SqlParameter("@Department", (object)employee.Department ?? DBNull.Value),
    new SqlParameter("@Salary", employee.Salary),
    new SqlParameter("@HireDate", employee.HireDate)
};

// 先執行 INSERT
// EF6 寫法 (GTS)
int rowsAffected = db.Database.ExecuteSqlCommand(insertSql, parameters.ToArray());

// EF Core 寫法 (Portal)
int rowsAffected = db.Database.ExecuteSqlRaw(insertSql, parameters.ToArray());

// 再取得最後插入的 ID
🚨 string getIdSql = "SELECT CAST(SCOPE_IDENTITY() as int)";

// EF6 寫法 (GTS)
int newId = db.Database.SqlQuery<🚨int>(getIdSql).First();

// EF Core 寫法 (Portal)  
int newId = db.Database.FromSqlRaw<int>(getIdSql).First();
```

**推薦：**

- 需要 ID → 使用**方法一**（OUTPUT 子句），效能較好且不會有併發問題
- 不需要 ID → 使用**方法二**，最簡單直接

### 3. 修改操作 (UPDATE)

**返回影響筆數，使用 ExecuteSqlCommand / ExecuteSqlRaw**

```csharp
string updateSql = @"
    UPDATE Employees 
    SET EmpName = @EmpName,
        Department = @Department,
        Salary = @Salary,
        HireDate = @HireDate
    WHERE EmpId = @EmpId";

var parameters = new List<SqlParameter>
{
    new SqlParameter("@EmpId", employee.EmpId),
    new SqlParameter("@EmpName", employee.EmpName),
    new SqlParameter("@Department", (object)employee.Department ?? DBNull.Value),
    new SqlParameter("@Salary", employee.Salary),
    new SqlParameter("@HireDate", employee.HireDate)
};

// EF6 寫法 (GTS)
int rowsAffected = db.Database.ExecuteSqlCommand(updateSql, parameters.ToArray());

// EF Core 寫法 (Portal)
int rowsAffected = db.Database.ExecuteSqlRaw(updateSql, parameters.ToArray());
```

### 4. 刪除操作 (DELETE)

**返回影響筆數，使用 ExecuteSqlCommand / ExecuteSqlRaw**

```csharp
string deleteSql = "DELETE FROM Employees WHERE EmpId = @EmpId";
var parameter = new SqlParameter("@EmpId", id);

// EF6 寫法 (GTS)
int rowsAffected = db.Database.ExecuteSqlCommand(deleteSql, parameter);

// EF Core 寫法 (Portal)
int rowsAffected = db.Database.ExecuteSqlRaw(deleteSql, parameter);
```

## 方法使用說明

### SqlQuery< T > / FromSqlRaw< T >（返回資料）

- 用途：需要返回資料集或特定值
- 會將查詢結果映射到指定的模型類別
- 適用於：
    - SELECT 查詢
    - INSERT 搭配 OUTPUT 取得新增的 ID

### ExecuteSqlCommand / ExecuteSqlRaw（返回影響筆數）

- 用途：需要返回執行結果（受影響筆數）的命令
- 返回 int 型別，表示受影響的資料筆數
- 適用於：
    - INSERT（不需要返回新 ID）
    - UPDATE
    - DELETE

## 注意事項

1. 使用查詢方法時，查詢結果的列必須能夠映射到模型類別的屬性
2. 處理可能為 null 的欄位時，建議使用 `(object)value ?? DBNull.Value`
3. 使用參數化查詢可以防止 SQL 注入攻擊
4. 使用 `OUTPUT INSERTED.欄位名` 可以在 INSERT 時直接取得新增的值
5. `SCOPE_IDENTITY()` <mark style="background: #FFF3A3A6;">只能取得當前連線最後一次插入的 ID</mark>，在併發環境下可能有風險

## 快速選擇指南

**系統選擇：**

- 在 GTS 系統開發時 → 使用 EF6 語法
- 在 Portal 系統開發時 → 使用 EF Core 語法

**方法選擇：**

- 需要返回資料 → SqlQuery / FromSqlRaw
- 只需要影響筆數 → ExecuteSqlCommand / ExecuteSqlRaw

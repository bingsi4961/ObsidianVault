---
title: 以 SQL 指令進行「新增、刪除、查詢、修改」
tags: [.Entity Framework]

---

# 查詢
```csharp=
public class EmployeeVM
{
    public int EmpId { get; set; }
    public string EmpName { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
    public DateTime HireDate { get; set; }
}

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
var emps = db.Database.SqlQuery<EmployeeVM>(sqlstr,params.ToArray()).ToList();
// emps => List<EmployeeVM>
```

# 新增
```csharp=
string insertSql = @"
    INSERT INTO Employees (EmpName, Department, Salary, HireDate)
    VALUES (@EmpName, @Department, @Salary, @HireDate);
    SELECT CAST(SCOPE_IDENTITY() as int)";

// 使用 SCOPE_IDENTITY() 取得新增資料的 ID

var parameters = new List<SqlParameter>
{
    new SqlParameter("@EmpName", employee.EmpName),
    new SqlParameter("@Department", (object)employee.Department ?? DBNull.Value),
    new SqlParameter("@Salary", employee.Salary),
    new SqlParameter("@HireDate", employee.HireDate)
};

// 執行插入並獲取新增的ID
int newId = db.Database.SqlQuery<int>(insertSql, parameters.ToArray()).First();
```

# 修改
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

// 回傳影響筆數
int rowsAffected = db.Database.ExecuteSqlCommand(updateSql, parameters.ToArray());
```

# 刪除
```csharp=
string deleteSql = "DELETE FROM Employees WHERE EmpId = @EmpId";
var parameter = new SqlParameter("@EmpId", id);

// 回傳影響筆數
int rowsAffected = db.Database.ExecuteSqlCommand(deleteSql, parameter);
```
1. `SqlQuery<T>`：
   - 用於需要返回**資料集**的查詢
   - 會將查詢結果映射到指定的模型類別
   - 典型用例：SELECT 查詢

2. `ExecuteSqlCommand`：
   - 用於需要返回**執行結果**（受影響筆數）的命令
   - 返回 int 型別，代表受影響的資料筆數
   - 典型用例：INSERT（不需要返回新 ID 時）、UPDATE、DELETE

---
在較新版本的 Entity Framework Core 中，這些方法已經改名為：
- `SqlQuery<T>` → `FromSqlRaw<T>`
- `ExecuteSqlCommand` → `ExecuteSqlRaw`
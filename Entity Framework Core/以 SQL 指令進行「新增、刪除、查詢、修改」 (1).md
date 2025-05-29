---
title: 以 SQL 指令進行「新增、刪除、查詢、修改」
tags: [.Entity Framework Core, 技術文件]

---

# 基本模型定義
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

# 常見操作範例

## 1. 查詢操作
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

var emps = db.Database.FromSqlRaw<EmployeeVM>(sqlstr, params.ToArray()).ToList();
```

## 2. 新增操作
```csharp
string insertSql = @"
    INSERT INTO Employees (EmpName, Department, Salary, HireDate)
    VALUES (@EmpName, @Department, @Salary, @HireDate);
    SELECT CAST(SCOPE_IDENTITY() as int)";

var parameters = new List<SqlParameter>
{
    new SqlParameter("@EmpName", employee.EmpName),
    new SqlParameter("@Department", (object)employee.Department ?? DBNull.Value),
    new SqlParameter("@Salary", employee.Salary),
    new SqlParameter("@HireDate", employee.HireDate)
};

// 執行插入並獲取新增的ID
int newId = db.Database.FromSqlRaw<int>(insertSql, parameters.ToArray()).First();
```

## 3. 修改操作
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
int rowsAffected = db.Database.ExecuteSqlRaw(updateSql, parameters.ToArray());
```

## 4. 刪除操作
```csharp
string deleteSql = "DELETE FROM Employees WHERE EmpId = @EmpId";
var parameter = new SqlParameter("@EmpId", id);

// 回傳影響筆數
int rowsAffected = db.Database.ExecuteSqlRaw(deleteSql, parameter);
```

# 方法使用說明

## FromSqlRaw<T>
- 用於需要返回資料集的查詢
- 會將查詢結果映射到指定的模型類別
- 適用於 SELECT 查詢

## ExecuteSqlRaw
- 用於需要返回執行結果（受影響筆數）的命令
- 返回 int 型別，表示受影響的資料筆數
- 適用於 INSERT（不需要返回新 ID 時）、UPDATE、DELETE 操作

# 注意事項
- 使用 FromSqlRaw 時，查詢結果的列必須能夠映射到模型類別的屬性
- 處理可能為 null 的欄位時，建議使用 DBNull.Value
- 使用參數化查詢可以防止 SQL 注入攻擊
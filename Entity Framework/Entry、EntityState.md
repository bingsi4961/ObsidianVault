---
title: Entry、EntityState
tags: [.Entity Framework]

---

# 說明 EntityState 的 Added、Modified、Deleted、Unchanged、Detached、Attach 

Entity Framework 中的 EntityState 代表實體物件在 DbContext 中的追蹤狀態，主要有以下幾種:

1. Added (新增)
- 代表這是一個新建立要插入資料庫的實體
- 使用情境:
```csharp=
var customer = new Customer { Name = "小明" };
context.Customers.Add(customer);
// 將 customer 的 EntityState = Added
// 開始追蹤這個實體 - 它會被加入到 ChangeTracker 中

// 或設定
context.Entry(customer).State = EntityState.Added;
// 如果 customer 還沒被追蹤，會將 customer 加入到 ChangeTracker 中
// 將 customer EntityState = Added

context.SaveChanges();
// INSERT INTO Customers (Name) VALUES ('小明')
```

2. Modified (修改)
- 代表這是一個已存在且被修改過的實體
- 使用情境:
```csharp=
var customer = context.Customers.Find(1); 
// 載入時 EntityState = Unchanged
// 開始追蹤這個實體 - 自動將實體加入 ChangeTracker

customer.Name = "小華";
// 修改後 EntityState = Modified

// 或設定
context.Entry(customer).State = EntityState.Modified;
// 如果 customer 還沒被追蹤，會將 customer 加入到 ChangeTracker 中
// 將 customer EntityState = Modified
// 此種寫法，在SaveChanges 時，會更新所有欄位

context.SaveChanges();
// UPDATE Customers SET Name = '小華' WHERE Id = 1
```

3. Deleted (刪除)
- 代表這個實體將要從資料庫中刪除
- 使用情境:
```csharp
var customer = context.Customers.Find(1);
// 載入時 EntityState = Unchanged
// 開始追蹤這個實體 - 自動將實體加入 ChangeTracker

context.Customers.Remove(customer);
// 如果 customer 還沒被追蹤，會將 customer 加入到 ChangeTracker 中
// 將 customer EntityState = Deleted

// 或設定
context.Entry(customer).State = EntityState.Deleted;
// 如果 customer 還沒被追蹤，會將 customer 加入到 ChangeTracker 中
// 將 customer EntityState = Deleted

context.SaveChanges();
// DELETE FROM Customers WHERE Id = 1
```

4. Unchanged (未變更)
- 代表實體從資料庫載入後沒有任何修改
- 當你從 DbContext 查詢資料時，預設狀態就是 Unchanged
```csharp
var customer = context.Customers.Find(1);
// 載入時 EntityState = Unchanged
// 開始追蹤這個實體 - 自動將實體加入 ChangeTracker

context.SaveChanges();
// 當執行 SaveChanges() 時，因為狀態是 Unchanged
// 不會產生任何 SQL，因為沒有變更需要儲存
```

5. Detached (分離)
- 代表實體沒有被 DbContext 追蹤
- 常見於從外部來源建立的實體物件
```csharp
var customer = new Customer { Id = 1, Name = "小明" };
// EntityState = Detached
// 此時實體不被 DbContext 追蹤，對資料庫沒有任何影響

var customers = context.Customers
    .AsNoTracking()
    .ToList();  // 所有查詢出來的實體都是 Detached 狀態
```

6. 關於 Attach 和 Detach 的操作:

- Attach (附加):
```csharp
// 如果 customer 還沒被追蹤，將 customer 加入到 ChangeTracker
// 設定狀態為 Unchanged
context.Customers.Attach(customer);
```

- Detach (分離):
```csharp
// 將 customer 實體從 ChangeTracker 中移除，不再追蹤這個實體。
context.Entry(customer).State = EntityState.Detached;
```

7. 實用提醒:
    * SaveChanges() 執行時會根據實體狀態，產生對應的 SQL 指令
    * 使用 Find() 或 LINQ 查詢時，實體會自動被追蹤
    * 如果使用 AsNoTracking()，查詢的實體就不會被追蹤    
    * SaveChanges() 會依照以下順序執行：
        - 先處理 INSERT (Added 狀態)
        - 再處理 UPDATE (Modified 狀態)
        - 最後處理 DELETE (Deleted 狀態)

8. Detached 轉換到其他狀態
```csharp
var customer = new Customer { Id = 1, Name = "小明" }; // Detached 狀態

// 1. 轉為 Added (新增)
context.Customers.Add(customer);

// 2. 轉為 Unchanged (附加)
context.Customers.Attach(customer);

// 3. 轉為 Modified (更新)
context.Entry(customer).State = EntityState.Modified;
```

- [使用實體狀態](https://learn.microsoft.com/zh-tw/ef/ef6/saving/change-tracking/entity-state?redirectedfrom=MSDN)
- [淺談 Entity Framework 的導覽屬性與外鍵的同步更新](https://hackmd.io/@CloudyWing/Bk7uSwL90#%E6%B7%BA%E8%AB%87-Entity-Framework-%E7%9A%84%E5%B0%8E%E8%A6%BD%E5%B1%AC%E6%80%A7%E8%88%87%E5%A4%96%E9%8D%B5%E7%9A%84%E5%90%8C%E6%AD%A5%E6%9B%B4%E6%96%B0)


# db.Products.Add(product) vs db.Products.Attach(product) 

讓我用簡單的方式解釋 `Add()` 和 `Attach()` 的差異：

1. `db.Products.Add(product)`：
```csharp
// 例如：新增一個產品
var newProduct = new Product { ProductName = "抗老眼霜", UnitPrice = 2180, CategoryID = 10 };
db.Products.Add(newProduct);
```
- 用於告訴 EF："這是一筆新的資料，請幫我新增到資料庫"
- EF 會將此實體標記為 ==EntityState.Added==
- 執行 SaveChanges() 時會產生 INSERT SQL 語句
- 適用於新資料（在資料庫中不存在的記錄）

2. `db.Products.Attach(product)`：
```csharp
// 例如：修改現有產品
var existingProduct = new Product { ProductID = 80, ProductName = "玫瑰精華液", UnitPrice = 1280, CategoryID = 10 };
db.Products.Attach(existingProduct);
db.Entry(existingProduct).State = EntityState.Modified;
```
- 用於告訴 EF："這筆資料在資料庫中已經存在，請開始追蹤它"
- EF 會將此實體標記為 ==EntityState.Unchanged==
- 通常搭配 `State = EntityState.Modified` 使用
- 執行 SaveChanges() 時會產生 UPDATE SQL 語句
- 適用於修改既有資料（在資料庫中已存在的記錄）

3. 實際應用的例子：
```csharp
using (var db = new NorthwindEntities())
{
    // 情境一：新增產品
    var newProduct = new Product { 
        ProductName = "新產品", 
        UnitPrice = 100 
    };
    db.Products.Add(newProduct);  // 會產生 INSERT 語句
    
    // 情境二：修改產品
    var existingProduct = new Product { 
        ProductID = 1,      // 已存在的 ID
        ProductName = "修改後的名稱", 
        UnitPrice = 200 
    };
    db.Products.Attach(existingProduct);
    db.Entry(existingProduct).State = EntityState.Modified;  // 會產生 UPDATE 語句
    
    db.SaveChanges();
}
```

4. 主要差異總結：
- `Add()`：
  * 用於新增資料
  * 不需要主鍵值（資料庫會自動產生）
  * 直接標記為 Added 狀態

- `Attach()`：
  * 用於修改資料
  * ==需要主鍵值（用來識別要修改的記錄）==
  * 預設標記為 Unchanged 狀態，通常需要手動設為 Modified

這就是為什麼在我們的程式中，根據 ProductID 是否為 0 來決定使用 Add 還是 Attach：
```csharp
if (product.ProductID == 0)
{
    // 新產品，沒有 ID，使用 Add
    db.Products.Add(product);
}
else
{
    // 既有產品，有 ID，使用 Attach 後設為 Modified
    db.Products.Attach(product);
    db.Entry(product).State = EntityState.Modified;
}
```


# 修改 Entity 屬性值的三種方法

1. **直接修改屬性**
```csharp
var existingCategory = context.Categories.Find(1);
existingCategory.Name = "New Name";
context.SaveChanges();
```
特點：
- ✅ 最簡單直觀的寫法
- ✅ EF 自動追蹤變更
- ✅ ==只更新被修改的欄位==
- ✅ 效能最好
- ✅ 最推薦的做法
- ⚠️ 需要注意實體是否被追蹤（若使用 AsNoTracking 則無法運作）

2. **設定 EntityState.Modified**
```csharp
var existingCategory = context.Categories.Find(1);
context.Entry(existingCategory).State = EntityState.Modified;
existingCategory.Name = "New Name";
context.SaveChanges();
```
特點：
- ❌ 會更新實體的所有欄位
- ❌ 效能較差
- ❌ 可能會覆蓋其他未修改的欄位
- ✅ 適合當你確定要更新所有欄位時使用
- ⚠️ 要小心使用，容易造成非預期的更新

3. **使用 SetValues**
```csharp
var category = new Category { Id = 1, Name = "New Name" };
var existingCategory = context.Categories.Find(1);
context.Entry(existingCategory).CurrentValues.SetValues(category);
context.SaveChanges();
```
特點：
- ✅ 只更新有設定值的欄位
- ✅ 適合用於 DTO 映射的情境
- ❌ 程式碼較複雜
- ❌ 需要建立新物件
- ⚠️ 預設不會更新導覽屬性

整體建議：
1. 一般情況下，優先使用「直接修改屬性」的方式
2. 如果是要從 DTO 更新資料，可以考慮使用 SetValues
3. 除非真的需要更新所有欄位，否則避免使用 EntityState.Modified
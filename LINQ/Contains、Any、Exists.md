---
title: Contains、Any、Exists
tags: [LINQ]

---

## Contains 方法

`Contains` 方法用於檢查集合中是否包含特定的元素。它返回一個布林值（`true` 或 `false`），表示元素是否存在於集合中。

### 基本語法

```csharp
bool result = collection.Contains(item);
```

### 實際範例

讓我們看一個簡單的例子，檢查一個整數陣列是否包含數字 5：

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7 };
bool containsFive = numbers.Contains(5); // 返回 true
bool containsTen = numbers.Contains(10); // 返回 false

Console.WriteLine($"陣列包含數字 5：{containsFive}");
Console.WriteLine($"陣列包含數字 10：{containsTen}");
```

當使用自定義類別時，我們可能需要提供一個比較器（comparer）來指定如何判斷兩個物件相等：

```csharp
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
}

// 自定義比較器
public class StudentComparer : IEqualityComparer<Student>
{
    public bool Equals(Student x, Student y)
    {
        return x.Id == y.Id;
    }

    public int GetHashCode(Student obj)
    {
        return obj.Id.GetHashCode();
    }
}

// 使用範例
List<Student> students = new List<Student>
{
    new Student { Id = 1, Name = "張三" },
    new Student { Id = 2, Name = "李四" },
    new Student { Id = 3, Name = "王五" }
};

Student searchStudent = new Student { Id = 2, Name = "不重要" };
bool containsStudent = students.Contains(searchStudent, new StudentComparer()); // 返回 true
```

## Any 方法

`Any` 方法更加靈活，它可以檢查集合中是否存在至少一個元素符合指定的條件。如果不提供條件，它會檢查集合是否包含任何元素（即集合是否為空）。

### 基本語法

```csharp
// 檢查集合是否包含任何元素
bool hasAnyElements = collection.Any();

// 檢查集合是否包含符合條件的元素
bool hasMatchingElement = collection.Any(element => condition);
```

### 實際範例

檢查集合是否為空：

```csharp
List<int> numbers = new List<int> { 1, 2, 3 };
bool hasElements = numbers.Any(); // 返回 true

List<int> emptyList = new List<int>();
bool isEmpty = emptyList.Any(); // 返回 false

Console.WriteLine($"numbers 集合包含元素：{hasElements}");
Console.WriteLine($"emptyList 集合包含元素：{isEmpty}");
```

檢查集合中是否有符合條件的元素：

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7 };

// 檢查是否有大於 5 的數字
bool hasNumberGreaterThanFive = numbers.Any(n => n > 5); // 返回 true

// 檢查是否有小於 0 的數字
bool hasNegativeNumber = numbers.Any(n => n < 0); // 返回 false

Console.WriteLine($"陣列包含大於 5 的數字：{hasNumberGreaterThanFive}");
Console.WriteLine($"陣列包含負數：{hasNegativeNumber}");
```

使用自定義類別的例子：

```csharp
List<Student> students = new List<Student>
{
    new Student { Id = 1, Name = "張三" },
    new Student { Id = 2, Name = "李四" },
    new Student { Id = 3, Name = "王五" }
};

// 檢查是否有 Id 為 2 的學生
bool hasStudentWithIdTwo = students.Any(s => s.Id == 2); // 返回 true

// 檢查是否有姓氏為「趙」的學生
bool hasStudentNamedZhao = students.Any(s => s.Name.StartsWith("趙")); // 返回 false

Console.WriteLine($"有 Id 為 2 的學生：{hasStudentWithIdTwo}");
Console.WriteLine($"有姓趙的學生：{hasStudentNamedZhao}");
```

## Contains 和 Any 的比較與選擇

雖然 `Contains` 和 `Any` 都用於檢查集合中的元素，但它們有不同的使用場景：

1. **簡單值比較**：如果只需檢查集合是否包含特定值，使用 `Contains`：
   ```csharp
   bool hasFive = numbers.Contains(5);
   ```

2. **複雜條件檢查**：如果需要檢查是否有元素符合特定條件，使用 `Any`：
   ```csharp
   bool hasEvenNumber = numbers.Any(n => n % 2 == 0);
   ```

3. **檢查集合是否為空**：使用不帶參數的 `Any` 方法：
   ```csharp
   bool isEmpty = !collection.Any();
   ```

4. **效能考量**：對於大型集合，`Contains` 通常比 `Any` 效能更好，因為 `Contains` 可能利用特定集合類型的優化（例如 HashSet 的快速查找）。但 `Any` 更具靈活性，可以處理複雜的條件。

## 綜合實例

讓我們看一個更複雜的例子，結合這兩個方法：

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
    public List<string> Tags { get; set; }
}

// 建立一些產品
List<Product> products = new List<Product>
{
    new Product 
    { 
        Id = 1, 
        Name = "筆記型電腦", 
        Category = "電子產品", 
        Price = 30000, 
        Tags = new List<string> { "電腦", "科技", "辦公" } 
    },
    new Product 
    { 
        Id = 2, 
        Name = "智慧型手機", 
        Category = "電子產品", 
        Price = 15000, 
        Tags = new List<string> { "手機", "科技", "通訊" } 
    },
    new Product 
    { 
        Id = 3, 
        Name = "書桌", 
        Category = "家具", 
        Price = 5000, 
        Tags = new List<string> { "家具", "辦公" } 
    }
};

// 使用 Contains：檢查是否有特定標籤的產品
List<string> searchTags = new List<string> { "科技", "家具" };
var productsWithTags = products.Where(p => p.Tags.Any(tag => searchTags.Contains(tag))).ToList();

Console.WriteLine("含有「科技」或「家具」標籤的產品：");
foreach (var product in productsWithTags)
{
    Console.WriteLine($"- {product.Name}");
}

// 使用 Any：檢查是否有價格超過 20000 的產品
bool hasExpensiveProduct = products.Any(p => p.Price > 20000);
Console.WriteLine($"有價格超過 20000 的產品：{hasExpensiveProduct}");

// 使用 Any 和 Contains 的組合：檢查「電子產品」類別中是否有含「辦公」標籤的產品
bool hasOfficeElectronics = products
    .Where(p => p.Category == "電子產品")
    .Any(p => p.Tags.Contains("辦公"));
Console.WriteLine($"電子產品類別中有含「辦公」標籤的產品：{hasOfficeElectronics}");
```

## 小結

`Contains` 和 `Any` 是 LINQ 中常用的兩個方法，掌握它們可以讓您的程式碼更簡潔、更具表達力：

- `Contains`：檢查集合中是否包含特定元素
- `Any`：檢查集合是否為空，或是否包含符合條件的元素

---

# C# LINQ 中的 Any 和 Exists 方法比較

## 基本概念與來源

### `List<T>.Exists`
- 屬於 `List<T>` 類的**實例方法**
- 定義在 `System.Collections.Generic` 命名空間中
- 來自 .NET 框架早期的集合類設計
- 僅適用於 `List<T>` 類型

### `IEnumerable<T>.Any`
- 屬於 LINQ 的**擴展方法**
- 定義在 `System.Linq` 命名空間中
- 代表更現代的函數式編程風格
- 適用於任何實現 `IEnumerable<T>` 介面的集合

## 功能與語法比較

### 相似之處
1. **功能目的**：兩者都用於檢查集合中是否存在符合條件的元素
2. **返回值**：兩者都返回布林值（true/false）
3. **使用委派**：兩者都接受一個謂詞函數作為條件判斷

### 關鍵差異
1. **無參數版本**：
   - `Any()` 有一個無參數版本，用於檢查集合是否包含任何元素
   - `Exists` 只有帶謂詞參數的版本

2. **適用範圍**：
   - `Any` 可用於所有實現 `IEnumerable<T>` 的集合（包括 Array、List、Dictionary 等）
   - `Exists` 只能用於 `List<T>` 物件

## 實際代碼示例

### 基本使用比較
```csharp
using System;
using System.Collections.Generic;
using System.Linq;

// 共用的陣列和列表
int[] numbersArray = { 1, 2, 3, 4, 5 };
List<int> numbersList = new List<int> { 1, 2, 3, 4, 5 };

// 使用 Any（適用於陣列和列表）
bool hasEvenNumberAny1 = numbersArray.Any(n => n % 2 == 0);  // 可用於陣列
bool hasEvenNumberAny2 = numbersList.Any(n => n % 2 == 0);   // 可用於列表

// 使用 Exists（僅適用於列表）
bool hasEvenNumberExists = numbersList.Exists(n => n % 2 == 0);  // 可用於列表

// 以下代碼會編譯錯誤
// bool error = numbersArray.Exists(n => n % 2 == 0);  // 無法用於陣列

Console.WriteLine($"陣列使用 Any 檢查偶數：{hasEvenNumberAny1}");
Console.WriteLine($"列表使用 Any 檢查偶數：{hasEvenNumberAny2}");
Console.WriteLine($"列表使用 Exists 檢查偶數：{hasEvenNumberExists}");
```

### Any 的擴展應用示例
```csharp
// 在複雜的 LINQ 查詢中使用 Any
var query = products
    .Where(p => p.Category == "電子產品")
    .Where(p => p.Tags.Any(t => t == "科技"));

// 使用 Any 檢查字典中的值
Dictionary<string, int> scores = new Dictionary<string, int> {
    { "張三", 85 },
    { "李四", 92 },
    { "王五", 78 }
};
bool hasHighScore = scores.Any(pair => pair.Value > 90);

// 使用無參數的 Any() 檢查集合是否為空
bool hasAnyRecords = customerList.Any();
```

### Exists 與其他 List 方法結合使用
```csharp
public class StudentProcessor {
    private List<Student> _students;
    
    public bool HasFailingStudent() {
        return _students.Exists(s => s.Score < 60);
    }
    
    public Student FindFirstFailingStudent() {
        return _students.Find(s => s.Score < 60);
    }
}
```

## 效能考量

在大多數情況下，兩者的效能差異很小，但在處理大型集合時可能會有所不同：

1. **Exists**：
   - 直接在 `List<T>` 上操作，不需要額外的迭代器
   - 對於 `List<T>` 類型，通常比 `Any` 稍快一些

2. **Any**：
   - 需要通過迭代器遍歷集合
   - 但它的優勢在於可以應用於任何 `IEnumerable<T>` 集合



## 實際應用場景選擇指南

### 選擇 Any 的情況
1. 當你需要處理各種不同類型的集合（陣列、字典等）
2. 當你在使用 LINQ 查詢鏈
3. 當你需要處理延遲執行（lazy evaluation）的情況
4. 處理 Entity Framework 或其他 ORM 的查詢
5. 需要檢查集合是否為空時（使用無參數版本）

### 選擇 Exists 的情況
1. 專門處理 `List<T>` 類型的集合
2. 注重效能，尤其是在大型列表上
3. 與其他 `List<T>` 特定方法（如 `Find`）一起使用
4. 不需要延遲執行，追求直接和明確的實現

## 兩者的設計理念差異

1. **Any**：
   - 代表了 LINQ 的設計理念：提供通用的集合查詢操作
   - 強調功能的一致性和表達能力
   - 支援查詢表達式和方法鏈

2. **Exists**：
   - 代表了集合類特定方法的設計思路
   - 與其他 `List<T>` 方法如 `Find`、`FindAll` 等形成一套完整的操作
   - 針對特定集合類型進行了優化

## 最佳實踐建議

在現代 C# 開發中，LINQ 已成為標準工具集的一部分，大多數開發者更傾向於使用 `Any`，因為：

1. **一致性**：它與其他 LINQ 方法風格一致
2. **通用性**：它可以應用於所有的集合類型
3. **可讀性**：LINQ 查詢通常被認為更具有聲明式編程的可讀性
4. **功能擴展**：與其他 LINQ 方法組合使用時非常強大

不過，如果你的代碼專門處理 `List<T>` 並且關注效能優化，則 `Exists` 可能是更好的選擇。

## 總結比較

| 特性 | Any | Exists |
|------|-----|--------|
| 適用範圍 | 任何 IEnumerable<T> | 僅 List<T> |
| 命名空間 | System.Linq | System.Collections.Generic |
| 無參數版本 | 有（檢查集合是否為空） | 無 |
| 執行方式 | 支援延遲執行 | 即時執行 |
| 效能 | 通用性高 | 在 List<T> 上更快 |
| 使用場景 | 多樣化集合處理和複雜查詢 | 專注於 List<T> 的高效操作 |
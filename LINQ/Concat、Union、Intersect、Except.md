---
title: Concat、Union、Intersect、Except
tags: [LINQ]

---

## 1. Concat (連接)

`Concat` 方法將兩個序列合併成單一序列，保留所有元素（包括重複項）。

### 工作原理
- 將第二個集合的所有元素添加到第一個集合的末尾
- 不會去除重複項
- 保持元素的原始順序

### 範例
```csharp
List<int> numbers1 = new List<int> { 1, 2, 3 };
List<int> numbers2 = new List<int> { 3, 4, 5 };

// 結果: { 1, 2, 3, 3, 4, 5 }
var concatenated = numbers1.Concat(numbers2);

foreach (var num in concatenated)
{
    Console.WriteLine(num);
}
```

在這個例子中，注意 `3` 出現了兩次，因為 `Concat` 不會去除重複的元素。

## 2. Union (聯集)

`Union` 方法將兩個序列合併成單一序列，但會去除重複項。它產生一個包含來自兩個集合的唯一元素的集合。

### 工作原理
- 合併兩個集合的元素
- 去除重複項（集合中每個元素只會出現一次）
- 使用默認的相等比較器來判斷元素是否相同

### 範例
```csharp
List<int> numbers1 = new List<int> { 1, 2, 3 };
List<int> numbers2 = new List<int> { 3, 4, 5 };

// 結果: { 1, 2, 3, 4, 5 }
var unionResult = numbers1.Union(numbers2);

foreach (var num in unionResult)
{
    Console.WriteLine(num);
}
```

在這個例子中，重複的 `3` 只出現一次，因為 `Union` 會去除重複項。

## 3. Intersect (交集)

`Intersect` 方法返回兩個序列中共有的元素（出現在兩個集合中的元素）。

### 工作原理
- 找出同時存在於兩個集合中的元素
- 去除重複項（結果集合中的每個元素只會出現一次）
- 使用默認的相等比較器來判斷元素是否相同

### 範例
```csharp
List<int> numbers1 = new List<int> { 1, 2, 3, 4 };
List<int> numbers2 = new List<int> { 3, 4, 5, 6 };

// 結果: { 3, 4 }
var intersectResult = numbers1.Intersect(numbers2);

foreach (var num in intersectResult)
{
    Console.WriteLine(num);
}
```

這個例子中，只有 `3` 和 `4` 同時出現在兩個集合中，所以它們是交集的結果。

## 4. Except (差集)

`Except` 方法返回第一個序列中存在但在第二個序列中不存在的元素。

### 工作原理
- 找出存在於第一個集合但不存在於第二個集合中的元素
- 去除重複項
- 使用默認的相等比較器來判斷元素是否相同


### 範例
```csharp
List<int> numbers1 = new List<int> { 1, 2, 3, 4 };
List<int> numbers2 = new List<int> { 3, 4, 5, 6 };

// 結果: { 1, 2 }
var exceptResult = numbers1.Except(numbers2);

foreach (var num in exceptResult)
{
    Console.WriteLine(num);
}
```

在這個例子中，`1` 和 `2` 只存在於 `numbers1` 中，不存在於 `numbers2` 中，所以它們是差集的結果。

## 自定義比較器

所有這些方法都可以使用自定義的 `IEqualityComparer<T>` 來比較元素。這在處理複雜對象時特別有用。

### 範例：使用自定義比較器
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

public class PersonComparer : IEqualityComparer<Person>
{
    public bool Equals(Person x, Person y)
    {
        return x.Name == y.Name;
    }

    public int GetHashCode(Person obj)
    {
        return obj.Name.GetHashCode();
    }
}

// 使用
List<Person> group1 = new List<Person> 
{
    new Person { Name = "Alice", Age = 25 },
    new Person { Name = "Bob", Age = 30 }
};

List<Person> group2 = new List<Person> 
{
    new Person { Name = "Alice", Age = 28 },
    new Person { Name = "Charlie", Age = 35 }
};

var collection = group1.Concat(group2);
// Name：Alice     Age：25
// Name：Bob       Age：30
// Name：Alice     Age：28
// Name：Charlie   Age：35

var collection = group1.Union(group2,new PersonComparer());
// Name：Alice     Age：25
// Name：Bob       Age：30
// Name：Charlie   Age：35

var collection = group1.Intersect(group2, new PersonComparer());
// Name：Alice     Age：25

var collection = group1.Except(group2, new PersonComparer());
// Name：Bob       Age：30


foreach (var item in collection)
{
    Console.WriteLine(item.Name + "/" + item.Age);
}
```

在這個例子中，我們創建了一個自定義的比較器，只根據名字來比較 `Person` 對象。即使兩個 Alice 的年齡不同，它們仍然被視為相同的元素。

## 集合操作的視覺化比較

為了更好地理解這四種方法的區別，我們可以用集合理論的角度來看：

- **Concat**: A + B（保留重複項）
- **Union**: A ∪ B（不保留重複項）
- **Intersect**: A ∩ B（只返回共有元素）
- **Except**: A - B（返回只在 A 中的元素）

## 應用場景示例

### Concat 適用於
- 需要保留所有元素（包括重複項）的場景
- 需要保持原始順序的場景
- 例如：合併來自多個數據源的日誌記錄

### Union 適用於
- 需要合併多個集合且不需要重複項的場景
- 例如：合併多個用戶列表，每個用戶只需出現一次

### Intersect 適用於
- 需要找出共同元素的場景
- 例如：找出同時喜歡兩種產品的用戶

### Except 適用於
- 需要找出差異的場景
- 例如：找出在 A 團隊但不在 B 團隊的成員
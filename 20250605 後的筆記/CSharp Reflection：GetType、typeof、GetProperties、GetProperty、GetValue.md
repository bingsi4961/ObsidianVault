---
date: 2025-07-16 14:56
aliases: 
tags:
  - CSharp_語法
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
#### 📑 [[]]

---
## 1. GetType() vs typeof() 比較

### GetType()

- 透過物件實例取得**實際執行時期**的型別資訊 ( <mark style="background: #FFF3A3A6;">物件實例</mark> )
- 會取得衍生類別的 Properties（如果物件是衍生類別實例）(包括父類別的 Properties)
- 需要物件 <mark style="background: #FFF3A3A6;">不能是 null</mark>，否則會拋出 `NullReferenceException`

### typeof()

- 取得**編譯時期**指定型別的 Properties
- 語法：`typeof(ClassName)`（注意：<mark style="background: #FFF3A3A6;">是型別名稱</mark>，不是變數名稱）
- 不需要建立物件實例

### 比較範例

```csharp
// 假設有以下類別結構
public class BaseModel
{
    public int Id { get; set; }
    public DateTime CreatedDate { get; set; }
}

public class DerivedModel : BaseModel
{
    public string Name { get; set; }
    public string Email { get; set; }
}

// 使用範例
BaseModel _derivedModel = new DerivedModel();

// 方法1：取得執行時期的實際型別 (DerivedModel)
var properties1 = _derivedModel.GetType().GetProperties(); 
// 結果：Id, CreatedDate, Name, Email

// 方法2：取得編譯時期的型別 (BaseModel)
var properties2 = typeof(BaseModel).GetProperties(); 
// 結果：Id, CreatedDate

// 方法3：取得指定型別的所有屬性 (DerivedModel)
var properties3 = typeof(DerivedModel).GetProperties(); 
// 結果：Id, CreatedDate, Name, Email（包含繼承的屬性）
```

## 2. GetValue() 方法

### 基本語法

```csharp
object value = propertyInfo.GetValue(objectInstance);
// object value = properties3[0].GetValue(_derivedModel);
```

### 功能

- 取得物件實例中特定屬性的值
- 回傳型別是 <mark style="background: #FFF3A3A6;">object</mark>，需要適當的型別轉換
- 如果屬性值是 `null`，會回傳 `null`

### 完整範例

```csharp
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Salary { get; set; }
}

// 使用範例
Employee emp = new Employee 
{ 
    Id = 1, 
    Name = "張三", 
    Salary = 50000 
};

// 取得所有屬性並顯示值
// properties 型態是 PropertyInfo[]
var properties = emp.GetType().GetProperties();

foreach (var prop in properties)
{
    object value = prop.GetValue(emp);
    Console.WriteLine($"{prop.Name}: {value}");
}
```

**輸出結果：**

```
Id: 1
Name: 張三
Salary: 50000
```

## 3. 常見應用場景

### 動態取值

<mark style="background: #FFF3A3A6;">使用 GetProperty</mark>，不是 GetProperties
```csharp
var nameProperty = typeof(Employee).GetProperty("Name");
string name = (string)nameProperty.GetValue(emp);
```

### 批次處理物件屬性

```csharp
public void PrintObjectProperties<T>(T obj)
{
    var properties = typeof(T).GetProperties();
    foreach (var prop in properties)
    {
        var value = prop.GetValue(obj);
        Console.WriteLine($"{prop.Name}: {value ?? "null"}");
    }
}
```

### 搭配 ClosedXML 使用（適用於 Portal 和 GTS 系統）

```csharp
// 動態將物件屬性寫入 Excel
var properties = rowModel.GetType().GetProperties();
for (int i = 0; i < properties.Length; i++)
{
    var value = properties[i].GetValue(rowModel);
    worksheet.Cell(row, i + 1).Value = value?.ToString() ?? "";
}
```

## 4. 重要注意事項

1. **型別轉換**：`GetValue()` 回傳 `object` 型別，使用時需要適當轉換
2. **Null 處理**：屬性值可能是 `null`，要做好 null 檢查
3. **效能考量**：Reflection 有效能成本，不要在高頻率執行的程式碼中過度使用
4. **錯誤處理**：注意 `NullReferenceException` 和型別轉換錯誤

## 5. 相關方法

- `SetValue(object, object)`：設定屬性值
- `GetProperties()`：取得所有公開屬性
- `GetProperty(string)`：取得指定名稱的屬性
- `DeclaringType`：取得宣告該屬性的型別
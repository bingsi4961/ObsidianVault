---
date: 2025-11-18 10:05
aliases:
tags:
  - CSharp_語法
  - JSON
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
## 一、前言：什麼是 JSON？

### 基本概念

JSON (JavaScript Object Notation) 是一種輕量級的資料交換格式，就像寄包裹時需要的「箱子和清單」。

### JSON 的兩種基本結構

1. **物件 (Object)**：用大括號 `{}` 包起來，裡面是成雙成對的「鍵 (Key)」和「值 (Value)」
2. **陣列 (Array)**：用中括號 `[]` 包起來，裡面是一連串有順序的值

### 生活範例

一張名片就可以看作是一個 JSON 物件：

```json
{
  "name": "王大明",
  "age": 30,
  "isMarried": true,
  "email": "david@example.com",
  "skills": ["C#", "Python", "SQL"]
}
```

## 二、JToken 家族樹與核心觀念

### JToken - 萬用容器（老祖宗）

- **概念**：所有 JSON 元素的共同祖先，代表 JSON 中的任何合法部分
- **比喻**：JSON 萬用容器、驚喜包、文章內容
- **特性**：是一個基底類別 (Base Class)

### JToken 的三個子類別（平輩兄弟姊妹）

#### 1. JObject（物件哥）

- **對應 JSON**：`{ }`
- **特徵**：專門用來表示「容器」，用 key 來裝 JToken
- **比喻**：字典、檔案櫃、一張名片

#### 2. JArray（陣列姐）

- **對應 JSON**：`[ ]`
- **特徵**：也是專門用來表示「容器」，用索引來裝 JToken
- **比喻**：列表、清單、購物清單

#### 3. JValue（單純弟）

- **對應 JSON**：字串 ("hi")、數字 (123)、布林 (true)、null
- **特徵**：專門用來表示「最終的值」，不能再裝任何東西
- **比喻**：一個單字、一個數字、一顆寶石

### 重要觀念釐清

- **JObject 和 JArray 是「容器型別」(Containers)**
- **JValue 是「終點型別」(Primitive / Leaf)**
- **JValue 絕對不會是 JObject 或 JArray**

## 三、JObject.Parse() - 解析的魔法

### 概念說明

- **Parse** 的英文原意是「解析」
- **作用**：讀取一段純文字，根據 JSON 文法規則，轉換成有結構的資料物件
- **完整意思**：「請你用 JSON 物件的文法規則，來解析這段文字，然後幫我建立一個對應的 JObject 物件」

### 為什麼需要 Parse？

- 沒有 Parse，C# 程式只會把 JSON 當成一般字串
- Parse 賦予文字結構和生命，讓它變成可以查詢、修改、遍歷的智慧型物件

### 生活比喻

就像拿到一張用日文寫的食譜（JSON 字串），請翻譯（Parse 方法）用中文告訴你內容，讓你真正「理解」和「使用」這份食譜。

### 基本使用範例

```csharp
using Newtonsoft.Json.Linq;
using System;

class Program
{
    static void Main(string[] args)
    {
        // 步驟 1: 取得原始的 JSON 文字資料
        string jsonString = @"
        {
          'name': '王大明',
          'age': 30,
          'isMarried': true,
          'email': 'david@example.com',
          'skills': ['C#', 'Python', 'SQL']
        }";

        // 步驟 2: 使用 JObject.Parse() 進行「解析」
        JObject personObject = JObject.Parse(jsonString);

        // 步驟 3: 像字典一樣，用「鍵」來取值
        string name = (string)personObject["name"];
        int age = (int)personObject["age"];

        // 步驟 4: 取出內部的陣列
        JArray skillsArray = (JArray)personObject["skills"];

        // 步驟 5: 遍歷 JArray
        foreach (var skill in skillsArray)
        {
            Console.WriteLine($"- {skill}");
        }
    }
}
```

### JArray.Parse() 的使用

如果 JSON 字串本身就是一個陣列：

```csharp
string jsonArrayString = "['蘋果', '香蕉', '橘子']";
JArray fruitArray = JArray.Parse(jsonArrayString);
```

## 四、JToken 的深入理解

### 為什麼需要 JToken？

因為 JSON 的結構是巢狀的，而且裡面的「值」型態不固定。

### JToken 作為「驚喜包」

當你使用中括號取值時：

```csharp
JObject person = JObject.Parse(jsonString);
JToken skillsToken = person["skills"];  // 回傳的是 JToken（驚喜包）
```

### 型別轉換（拆箱）

```csharp
// 拆箱！我斷言這裡面一定是 JArray
JArray skillsArray = (JArray)skillsToken;

// 或者合併寫法
JArray skillsArray = (JArray)person["skills"];
```

### 使用 is 搭配模式匹配（Pattern Matching）進行型別判斷與轉換

C# 7 開始支援 `is` 搭配模式匹配（Pattern Matching），可以「判斷型別 + 宣告變數」一步完成，比傳統的先判斷再轉型更簡潔安全：

```csharp
// ✅ C# 7+ 模式匹配寫法：判斷 + 轉型一步完成
if (item is JObject jObject)
{
    // jObject 已經是 JObject 型別，可以直接使用
    Console.WriteLine($"找到物件，裡面有 {jObject.Count} 個屬性");
    string task = (string)jObject["task"];
}

// ❌ 傳統寫法：需要兩步驟
if (item is JObject)
{
    JObject jObject = (JObject)item;  // 還要再轉型一次
    Console.WriteLine($"找到物件，裡面有 {jObject.Count} 個屬性");
}
```

> **小提醒**：模式匹配的變數 `jObject` 只在 `if` 區塊內有效（作用域限於該區塊），且如果型別不符就不會進入區塊，因此不會有 `InvalidCastException` 的風險。

### JToken.Type 屬性

可以用來檢查 JToken 的實際類型：

```csharp
foreach (JToken item in mixedArray)
{
    Console.WriteLine($"Token 型別: {item.Type}");
    
    if (item.Type == JTokenType.String)
    {
        Console.WriteLine($"  我是一個字串: {(string)item}");
    }
}
```

### 使用 JToken.Type 搭配 switch 做分流處理

當 JToken 的型別有多種可能時，使用 `switch` 搭配 `JTokenType` 列舉來做分流處理，特別適合需要「針對不同型別做不同渲染或格式化」的場景：

```csharp
/// <summary>
/// 根據 JToken 的型別，分流處理並回傳對應的字串結果
/// </summary>
private string RenderToken(JToken token)
{
    switch (token.Type)
    {
        case JTokenType.Object:
            // 遇到物件，委派給專門處理物件的方法
            return RenderObject((JObject)token);

        case JTokenType.Array:
            // 遇到陣列，委派給專門處理陣列的方法
            return RenderArray((JArray)token);

        case JTokenType.Null:
        case JTokenType.Undefined:
            // null 或 undefined 直接回傳空字串
            return string.Empty;

        default:
            // 其餘（String、Integer、Float、Boolean 等 JValue）
            // 直接轉成字串輸出
            return Encode(token.ToString());
    }
}
```

> **重點整理**：
> - `JTokenType.Object` 和 `JTokenType.Array` 是容器型別
> - `JTokenType.Null` 和 `JTokenType.Undefined` 建議特別處理，避免 `NullReferenceException`
> - `default` 涵蓋所有 JValue 的子型別（String、Integer、Float、Boolean、Date 等）

## 五、JValue 的實戰使用

### 情境一：手動建立 JSON 時處理 null 值

```csharp
JObject newPerson = new JObject();

// 錯誤做法：C# 的 null 不等於 JSON 的 null
newPerson["nickname"] = null;  // nickname 欄位不會被加入

// 正確做法：明確建立 JValue 表示 JSON 的 null
newPerson["nickname"] = JValue.CreateNull();
// 或者
newPerson["nickname"] = new JValue(null);
```

### 情境二：檢查不確定的 JSON 內容

```csharp
JArray mixedArray = JArray.Parse(jsonString);

foreach (JToken item in mixedArray)
{
    if (item is JValue)
    {
        // 處理單純的值
        JValue simpleValue = (JValue)item;
        Console.WriteLine($"[單純的值]: {simpleValue.Value}");
    }
    else if (item is JObject)
    {
        // 處理物件
        Console.WriteLine($"[複雜的物件]: {item.ToString()}");
    }
}
```

## 六、JProperty - 鍵值對的表示

### 核心概念

- **JProperty 代表一個「鍵值對」**
- **JObject 是 JProperty 的集合**

### JProperty 的結構

1. **Name（鑰匙/鍵）**：字串類型
2. **Value（驚喜包/值）**：JToken 類型

### 實戰使用情境

#### 情境一：遍歷 JObject

```csharp
JObject person = JObject.Parse(jsonString);

foreach (JProperty prop in person.Properties())
{
    // 取得鍵
    Console.WriteLine($"鍵 (Key): {prop.Name}");
    
    // 取得值（JToken）
    JToken valueToken = prop.Value;
    Console.WriteLine($"  值的型別: {valueToken.Type}");
    Console.WriteLine($"  值的內容: {valueToken.ToString()}");
}
```

#### 情境二：手動建立 JObject

```csharp
// 建立 JProperty
JProperty nameProperty = new JProperty("name", "王大明");
JProperty skillsProperty = new JProperty("skills", 
                                new JArray("C#", "SQL"));

// 建立 JObject 並加入 JProperty
JObject person = new JObject();
person.Add(nameProperty);
person.Add(skillsProperty);
```

### 安全取值：TryGetValue 方法

使用中括號 `jObject["key"]` 取值時，如果 key 不存在會回傳 `null`，後續操作可能觸發 `NullReferenceException`。
更安全的做法是使用 `TryGetValue`，它會先檢查 key 是否存在，存在才取值：

```csharp
JObject person = JObject.Parse(jsonString);

// ✅ 安全做法：TryGetValue 先檢查再取值
if (person.TryGetValue("email", out JToken emailToken))
{
    // key 存在，emailToken 已經有值，可以安全使用
    string email = emailToken.ToString();
    Console.WriteLine($"Email: {email}");
}
else
{
    // key 不存在，不會拋出例外
    Console.WriteLine("找不到 email 欄位");
}

// ❌ 不安全做法：如果 "phone" 不存在，後續 .ToString() 會爆炸
// string phone = person["phone"].ToString();  // NullReferenceException!
```

> **TryGetValue vs 中括號取值的比較**：
> - `jObject["key"]`：key 不存在時回傳 `null`，需要自己做 null 檢查
> - `jObject.TryGetValue("key", out JToken value)`：回傳 `bool`，成功時 `value` 已填入，失敗時不會出錯
> - 當你不確定 JSON 是否包含某個欄位時（例如處理外部 API 回傳的資料），**強烈建議使用 TryGetValue**

## 七、JArray 的彈性 - 混合型態陣列

### 重要觀念

**JArray 可以包含 JObject、JArray 和 JValue**，因為它們都是 JToken。

### 範例：大雜燴陣列

```json
[
  "這是一個 JValue (字串)",
  123,
  {
    "id": 1,
    "task": "這是一個 JObject"
  },
  [
    "紅色",
    "藍色",
    "這是一個巢狀 JArray"
  ],
  true,
  null
]
```

### 處理混合型態陣列

```csharp
JArray mixedArray = JArray.Parse(jsonString);

foreach (JToken item in mixedArray)
{
    if (item is JValue)
    {
        Console.WriteLine($"  -> 內容 (JValue): {item}");
    }
    else if (item is JObject)
    {
        JObject obj = (JObject)item;
        Console.WriteLine($"  -> 內容 (JObject): 找到任務 '{obj["task"]}'");
    }
    else if (item is JArray)
    {
        JArray arr = (JArray)item;
        Console.WriteLine($"  -> 內容 (JArray): 裡面有 {arr.Count} 個元素");
    }
}
```

### 使用 LINQ 快速判斷 JArray 是否全為基本型別

有時候我們需要根據 JArray 的內容來決定處理方式，例如：全部都是基本型別（字串、數字等）時直接輸出，有巢狀物件或陣列時則需要遞迴處理。
可以用 LINQ 的 `All` 方法搭配 `JTokenType` 來快速判斷：

```csharp
JArray array = JArray.Parse(jsonString);

// 檢查陣列中是否全部都是「基本型別」（不含 Object 和 Array）
bool allPrimitive = array.All(t =>
    t.Type != JTokenType.Object && t.Type != JTokenType.Array);

if (allPrimitive)
{
    // 全部都是 JValue（字串、數字、布林、null 等），可以直接逐一輸出
    string result = string.Join(", ", array.Select(t => t.ToString()));
    Console.WriteLine($"簡單陣列: [{result}]");
}
else
{
    // 裡面包含 JObject 或 JArray，需要遞迴或特殊處理
    Console.WriteLine("這是一個包含巢狀結構的複雜陣列");
}
```

## 八、ToObject - 從動態到靜態的轉換

### 概念說明

- **ToObject 把 JObject/JArray 自動轉換成預先定義好的 C# 實體模型**
- **從「動態」走向「靜態」、從「靈活」走向「安全」**

### 為什麼需要 ToObject？

手動取值的缺點：

1. **繁瑣**：多個欄位要寫多行取值和轉型
2. **危險**：轉型錯誤會在執行時崩潰
3. **難用**：得到零散的變數而非完整物件

### 使用步驟

#### 步驟 1：定義 C# 類別 (POCO)

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public bool IsMarried { get; set; }
    public List<string> Skills { get; set; }
}
```

#### 步驟 2：解析並使用 ToObject

```csharp
// 解析成 JObject
JObject jObject = JObject.Parse(jsonString);

// 使用 ToObject 自動對應到 Person 類別
Person personObject = jObject.ToObject<Person>();

// 享受強型別的好處
Console.WriteLine($"姓名: {personObject.Name}");
Console.WriteLine($"年齡: {personObject.Age}");
```

### JArray 也可以使用 ToObject

```csharp
string jsonArrayString = "['蘋果', '香蕉', '橘子']";
JArray jArray = JArray.Parse(jsonArrayString);

// 轉換成 List<string>
List<string> fruitList = jArray.ToObject<List<string>>();
```

## 九、兩種處理方式的比較

| 特性       | **JObject / JToken (手動解析)**        | **ToObject<T> (自動對應)**                           |
| -------- | ---------------------------------- | ------------------------------------------------ |
| **比喻**   | 靈活的地圖                              | 精確的模型                                            |
| **用法**   | `(string)jObject["key"]`           | `jObject.ToObject<MyClass>()`                    |
| **優點**   | 1. 靈活：JSON 結構不固定時<br>2. 快速：只取部分欄位時 | 1. 安全：強型別，編譯期檢查<br>2. 方便：自動填充所有欄位<br>3. 易讀：程式碼乾淨 |
| **缺點**   | 1. 繁瑣：要寫一堆轉型<br>2. 危險：執行期轉型可能出錯    | 1. 死板：JSON 必須符合 Class 結構<br>2. 準備：必須先定義 C# Class |
| **使用時機** | 不確定 JSON 結構，或只想修改/查詢一小部分           | 明確知道 API 回傳的 JSON 結構，且要使用大部分資料                   |

## 十、總結

### 類別總覽表

| 類別        | 比喻            | 核心概念                         | 主要用途             |
| --------- | ------------- | ---------------------------- | ---------------- |
| JToken    | JSON 萬用容器、驚喜包 | 所有 JSON 元素的「共同祖先」            | 表示 JSON 中的任何合法部分 |
| JObject   | 字典、檔案櫃        | 一種 JToken，專門表示 {}            | 用「鍵」來查找資料        |
| JArray    | 購物清單、列表       | 一種 JToken，專門表示 []            | 用「索引」來存取或遍歷項目    |
| JValue    | 單字、數字、寶石      | 一種 JToken，表示最終的值             | 存放字串、數字、布林值等     |
| JProperty | 字典條目          | JObject 專用，由 Name + Value 組成 | 代表一個鍵值對          |

### 重要觀念回顧

1. **JObject 是 JProperty 的集合**
2. **JArray 是 JToken 的集合（可包含任何 JToken）**
3. **JValue 只能存放最終的值，不能是容器**
4. **Parse 方法將 JSON 字串轉換為可操作的物件**
5. **ToObject 方法將 JToken 轉換為強型別的 C# 物件**
6. **使用 `is` 模式匹配可以同時判斷型別並轉型，比傳統 `is` + 強制轉型更安全簡潔**
7. **使用 `TryGetValue` 安全取值，避免因 key 不存在而拋出例外**
8. **使用 `JToken.Type` 搭配 `switch` 可以清晰地分流處理不同型別的 JSON 元素**

在現代 C# 開發中，通常稱 JObject 的方式為「動態解析 (Dynamic Parsing)」，而 ToObject 的方式為「反序列化 (Deserialization)」。90% 的情況下，ToObject 都是首選方法，因為它安全、乾淨且效率極高。
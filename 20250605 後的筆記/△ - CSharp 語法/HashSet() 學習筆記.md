---
date: 2025-12-04 12:05
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

---
## 一、HashSet\<T> 是什麼？

HashSet\<T> 是一個專門用來儲存**不重複元素**的集合，底層實作基於**雜湊表 (Hash Table)**。

### 核心特性

1. **唯一性 (Uniqueness)**：自動過濾重複項目    
    - 嘗試加入已存在的元素時，`Add()` 會回傳 `false`
    - 不會拋出錯誤，也不會加入該元素

2. **無序性 (Unordered)**：不保證元素順序，沒有索引 (Index)
    
3. **極致效能**：    
    - 搜尋 (`Contains`)、新增 (`Add`)、移除 (`Remove`) 的時間複雜度平均為 **O(1)**
    - 相比 `List<T>` 的 `Contains` 為 O(n)，當資料量達到萬筆以上時效能差異巨大

---
## 二、基本操作

```csharp
using System;
using System.Collections.Generic;

var techStack = new HashSet<string>();

// 1. 新增元素
techStack.Add("C#");
techStack.Add("MVC");
techStack.Add(".Net Core");

// 嘗試新增重複元素
bool isAdded = techStack.Add("C#"); // 回傳 false (重複)

// 2. 移除元素
techStack.Remove("MVC");

// 3. 判斷是否存在 (這是 HashSet 最強的地方)
if (techStack.Contains(".Net Core"))
{
    Console.WriteLine("技術堆疊包含 .Net Core");
}

// 4. 取得數量
Console.WriteLine($"目前元素數量: {techStack.Count}");

// 5. 走訪 (因為沒有索引，必須用 foreach)
foreach (var tech in techStack)
{
    Console.WriteLine(tech);
}
```

---
## 三、集合運算 (Set Operations)

**適用場景**：比對兩組資料（例如：比對「使用者現有權限」與「系統所有權限」）

假設我們有兩個集合：

- **Set A** (User A 的技能): C#, SQL, JavaScript
- **Set B** (User B 的技能): SQL, Python, HTML

```csharp
var setA = new HashSet<string> { "C#", "SQL", "JavaScript" };
var setB = new HashSet<string> { "SQL", "Python", "HTML" };

// 複製一份以免影響原始資料
var result = new HashSet<string>(setA);

// 1. 聯集 (Union) - A + B (取所有不重複的)
result.UnionWith(setB); 
// 結果: C#, SQL, JavaScript, Python, HTML

// 2. 交集 (Intersect) - A & B (兩者都有的)
result = new HashSet<string>(setA); // 重置
result.IntersectWith(setB);
// 結果: SQL

// 3. 差集 (Except) - A - B (A 有但 B 沒有的)
result = new HashSet<string>(setA); // 重置
result.ExceptWith(setB);
// 結果: C#, JavaScript (SQL 被扣掉了)

// 4. 對稱差集 (Symmetric Except) - (A + B) - (A & B) (彼此獨有的)
result = new HashSet<string>(setA); // 重置
result.SymmetricExceptWith(setB);
// 結果: C#, JavaScript, Python, HTML (排除了共同擁有的 SQL)
```

---
## 四、自定義物件的正確使用方式（重點章節）

### 4.1 為什麼需要特別處理？核心問題說明

在 C# 中，`class` (類別) 是**參考型別 (Reference Type)**。

預設情況下，C# 判斷兩個物件是否相等的標準是：**「它們是不是記憶體中的同一個位址？」**

#### 用識別證來比喻

想像你在華碩辦公室：

- 你拿了一張「**識別證 A**」，上面寫著工號 001、姓名 Alex
- 人資又印了一張「**識別證 B**」，上面也寫著工號 001、姓名 Alex

對 C# 預設來說：

- 這是**兩張不同的塑膠卡片**（不同的記憶體位址）
- 所以 HashSet 會覺得它們是**兩個不一樣的東西**
- 導致你的集合裡出現**兩個 Alex**

但對你的系統來說：

- **工號 (ID) 一樣就是同一個人**

**IEquatable\<User> 這段程式碼，就是在教 C# 改變判斷標準：不要管卡片是不是同一張，請去檢查上面的「工號」一不一樣。**

---
### 4.2 錯誤示範

```csharp
// ❌ 錯誤示範：沒有實作 IEquatable
public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}

var users = new HashSet<User>();
users.Add(new User { Id = 1, Name = "Alex" });
users.Add(new User { Id = 1, Name = "Alex" }); 

Console.WriteLine(users.Count); 
// 輸出: 2 ❌
// 因為這是兩個不同的物件實體 (new 兩次)
// C# 預設比較記憶體位址，發現是兩個不同位址
```

---
### 4.3 正確做法：實作 IEquatable\<T>

```csharp
public class User : IEquatable<User>
{
    public int Id { get; set; }
    public string Name { get; set; }

    // 1. 實作 Equals 來定義「什麼叫做相等」
    public bool Equals(User other)
    {
        // 如果對方是 null (空值)，當然不相等，回傳 false
        if (other is null) return false;
        
        // 這裡是關鍵！
        // 我們告訴電腦：只要 "這一個 Id" 等於 "那一個 Id"，就是同一個人
        return this.Id == other.Id;
    }

    // 2. 必須覆寫 GetHashCode (這是 HashSet 運作的關鍵)
    // 相同的物件必須回傳相同的 HashCode
    public override int GetHashCode()
    {
        // 因為我們是用 Id 來判斷是否同一個人
        // 所以我們直接回傳 Id 的雜湊碼當作分類依據
        // 這樣 ID=1 的物件，永遠會拿到一樣的 HashCode
        return this.Id.GetHashCode();
    }

    // 3. 為了完整性，通常也會覆寫 object 的 Equals
    public override bool Equals(object obj) => Equals(obj as User);
}

// 實作後：
var users = new HashSet<User>();
users.Add(new User { Id = 1, Name = "Alex" });
users.Add(new User { Id = 1, Name = "Alex" });

Console.WriteLine(users.Count); 
// 輸出: 1 ✓
// 成功過濾重複
```

---
## 五、深入理解三個方法的作用

### 5.1 方法一：`Equals(User other)` - VIP 通道（高效能）

```csharp
public bool Equals(User other)
{
    // 如果對方是 null (空值)，當然不相等，回傳 false
    if (other is null) return false;

    // 這裡是關鍵！
    // 我們告訴電腦：只要 "這一個 Id" 等於 "那一個 Id"，就是同一個人
    return this.Id == other.Id;
}
```

#### 作用說明

這是合約中規定的方法，這是真正執行「比對」的地方。

- **目的**：定義「什麼叫做相等」的標準
- **使用時機**：當變數型別明確是 `User` 時，編譯器會優先呼叫這個方法
- **效能**：最快（不需要轉型，不需要 Boxing/Unboxing）
- **給誰用**：給泛型集合如 `HashSet<User>`、`List<User>` 使用

#### 為什麼叫 VIP 通道？

因為編譯器「知道」這是 `User` 類型，可以直接呼叫專屬的比較方法，不需要經過任何轉換。

---
### 5.2 方法二：`GetHashCode()` - 分類依據（效能關鍵）

```csharp
public override int GetHashCode()
{
    // 因為我們是用 Id 來判斷是否同一個人
    // 所以我們直接回傳 Id 的雜湊碼當作分類依據
    // 這樣 ID=1 的物件，永遠會拿到一樣的 HashCode
    return this.Id.GetHashCode();
}
```

#### 作用說明

這是 **HashSet 運作的核心**，也是最容易忘記的！

HashSet 為了效能極快 (O(1))，它不會一開始就去比對 ID（因為太慢）。它會先把物件分類到不同的「桶子」裡。這個「桶子」的編號就是 **HashCode**。

---
#### 運作流程（兩階段檢查）

**步驟一：先看 GetHashCode()**

- HashSet 先看 `GetHashCode()`
- 如果兩個物件的 HashCode **不同**，它直接認定這兩個是不一樣的（連 ID 都不用比了）
- 這樣可以極快速地排除大部分不相同的物件

**步驟二：HashCode 相同時，才呼叫 Equals()**

- 如果 HashCode **相同**，它才會進一步呼叫上面的 `Equals()` 來確認 ID 真的相同嗎
- 這是為了處理**雜湊碰撞 (Hash Collision)** 的情況

---
#### 圖解流程

假設你現在要把 `User { Id = 1 }` 加入 HashSet：

```
1. HashSet 問：「這個新物件的 GetHashCode 是多少？」
   └─ 回答：是 1001 (假設)

2. HashSet 看：「我的 1001 號桶子裡有沒有東西？」
   
   情況 A (沒東西)：
   └─ 直接放入 (不用比對 ID，速度極快)
   
   情況 B (有東西)：
   └─ 裡面已經有一個物件了，它的 HashCode 也是 1001
   
3. HashSet 再問 (遇到碰撞)：
   「既然都在同一個桶子，那我呼叫 Equals() 比對一下它們的 ID 是一樣的嗎？」
   
   └─ 回答 True：重複了！丟棄新物件
   └─ 回答 False：原來只是剛好分到同組，放入桶子
```

---
#### 重要原則

**相同的物件必須回傳相同的 HashCode**

如果你的 `Equals()` 判斷兩個物件相等，那它們的 `GetHashCode()` 就**必須**回傳相同的值。

```csharp
User u1 = new User { Id = 1 };
User u2 = new User { Id = 1 };

// 如果 u1.Equals(u2) 為 true
// 那麼 u1.GetHashCode() 必須等於 u2.GetHashCode()
```

---
#### 效能陷阱警告

如果 `GetHashCode` 實作很爛（例如全部回傳常數 1）：

```csharp
// ❌ 糟糕的實作
public override int GetHashCode()
{
    return 1; // 所有物件都回傳 1
}
```

結果：

- 所有物件都會被丟到同一個桶子裡
- HashSet 會退化成**鏈結串列 (Linked List)**
- 效能從 **O(1) 變回 O(n)**
- 完全失去使用 HashSet 的意義

---
### 5.3 方法三：`override Equals(object obj)` - 一般通道（相容性）

```csharp
public override bool Equals(object obj)
{
    // 嘗試把傳進來的東西轉成 User
    // 如果轉型成功就呼叫上面寫好的那個 Equals
    return Equals(obj as User);
}
```

#### 作用說明

這是一個**保險機制**，確保相容性。

- **目的**：當物件被轉型成 `object` 時，仍能正確比較
- **使用時機**：舊系統、非泛型集合、或被轉型成 `object` 的情況
- **實作方式**：把「一般通道」的請求，轉發給「VIP 通道」處理

---
#### 為什麼需要這個方法？

`IEquatable<User>` 是給「知道它是 User 的人」用的（VIP 通道）；而 `object.Equals` 是給「不知道它是什麼，只知道它是個物件的人」用的（一般通道）。

如果不覆寫 `object.Equals`，當你的物件被當作「普通物件 (object)」來操作時，你的比較邏輯就會失效。

---
#### 實際範例：說明為什麼重要

```csharp
User u1 = new User { Id = 1 };
User u2 = new User { Id = 1 };

// 情境 A：變數型別明確是 User
// 編譯器知道它是 User，所以會走 IEquatable 的 VIP 通道
bool result1 = u1.Equals(u2); 
// 👉 結果：True ✓ (正確，因為用了你的邏輯)

// 情境 B：變數被轉型成 object (這在系統底層很常見！)
object o1 = u1;
object o2 = u2;

// 編譯器現在只把 o1 當作普通 object
// 它不知道有 VIP 通道，所以它去呼叫最原始的 object.Equals
bool result2 = o1.Equals(o2); 
// 👉 如果沒有覆寫：結果會是 False ❌ (錯誤！)
// 因為它退化成比較「記憶體位址」，發現是兩個不同物件，就回傳 False

// 👉 如果有覆寫：結果會是 True ✓ (正確！)
// 因為我們把請求轉發給 VIP 通道處理
```

---
#### 什麼時候會遇到「被轉型成 object」的情況？

在 .NET Framework 或許多第三方套件的底層，很多時候為了通用性，程式碼是這樣寫的：

```csharp
// 底層套件的程式碼
public void CheckSomething(object itemA, object itemB)
{
    // 底層套件不知道你是 User，它只把你當 object
    if (itemA.Equals(itemB)) 
    {
        // 執行某些邏輯...
    }
}

// 你的程式碼
User user1 = new User { Id = 1 };
User user2 = new User { Id = 1 };

// 當你呼叫這個方法時，User 會被自動轉型成 object
CheckSomething(user1, user2);
```

如果你沒有覆寫 `object.Equals`，在這種情況下：

- 你的「ID 相同視為同一人」的規則就會**失效**
- 導致系統出現奇怪的 Bug（明明 ID 一樣，系統卻說是不同的東西）

---
#### 總結：兩個通道的比喻

```
IEquatable<User>.Equals(User other)  →  VIP 通道（快速、高效）
         ↑
         |
    知道型別是 User
         |
    
object.Equals(object obj)  →  一般通道（相容、安全）
         ↑
         |
    只知道是 object
         |
    內部轉發給 VIP 通道處理
```

身為華碩的資深工程師，寫出這兩個方法的組合拳，代表你的程式碼：

- **既有效能** (`IEquatable<T>`)
- **又有強健性** (`override object.Equals`)

這是非常專業的寫法。

---
## 六、完整範例：三個方法如何協同運作

```csharp
public class User : IEquatable<User>
{
    public int Id { get; set; }
    public string Name { get; set; }

    // VIP 通道：給泛型集合使用，效能最好
    public bool Equals(User other)
    {
        if (other is null) return false;
        return this.Id == other.Id;
    }

    // 分類依據：決定物件放在哪個桶子
    public override int GetHashCode()
    {
        return this.Id.GetHashCode();
    }

    // 一般通道：確保相容性，轉發給 VIP 通道
    public override bool Equals(object obj)
    {
        return Equals(obj as User);
    }
}
```

### 測試流程說明

```csharp
var users = new HashSet<User>();

User alex1 = new User { Id = 1, Name = "Alex" };
User alex2 = new User { Id = 1, Name = "Alex" };
User bob = new User { Id = 2, Name = "Bob" };

// 第一次加入 alex1
users.Add(alex1);
// HashSet 流程：
// 1. 呼叫 alex1.GetHashCode() → 假設得到 1001
// 2. 檢查 1001 號桶子 → 空的
// 3. 放入 alex1

// 第二次加入 alex2（ID 相同）
users.Add(alex2);
// HashSet 流程：
// 1. 呼叫 alex2.GetHashCode() → 也得到 1001（因為 Id 相同）
// 2. 檢查 1001 號桶子 → 裡面有 alex1
// 3. 呼叫 alex1.Equals(alex2) → 回傳 true
// 4. 判定重複，不加入

// 第三次加入 bob（ID 不同）
users.Add(bob);
// HashSet 流程：
// 1. 呼叫 bob.GetHashCode() → 得到 2002
// 2. 檢查 2002 號桶子 → 空的
// 3. 放入 bob

Console.WriteLine(users.Count); // 輸出: 2 ✓
```

---
## 七、效能比較：HashSet vs List

| 特性   | List<T>        | HashSet<T>          | 使用建議               |
| ---- | -------------- | ------------------- | ------------------ |
| 資料結構 | 動態陣列 (Array)   | 雜湊表 (Hash Table)    | -                  |
| 重複資料 | 允許             | 不允許                 | 需要唯一值選 HashSet     |
| 有序性  | 有序（有 Index）    | 無序                  | 需要用 Index 存取選 List |
| 搜尋速度 | O(n)（隨資料量線性增加） | O(1)（極快，與資料量無關）     | 大量資料比對選 HashSet    |
| 記憶體  | 較省             | 較耗（需維護 Hash bucket） | -                  |
|      |                |                     |                    |

---
### 實務應用場景

**Portal 系統範例**：從資料庫撈出 10 萬筆 Log，過濾出其中有哪些獨特的 UserId

```csharp
// 方法一：使用 List (效能較差)
var userIdList = new List<int>();
foreach (var log in logs) // 假設有 10 萬筆
{
    userIdList.Add(log.UserId);
}
var uniqueUserIds = userIdList.Distinct(); // 最後再去重複

// 方法二：使用 HashSet (效能最佳)
var userIdSet = new HashSet<int>();
foreach (var log in logs) // 假設有 10 萬筆
{
    userIdSet.Add(log.UserId); // 自動去重複
}
// 後續查詢該 User 是否存在時極快
if (userIdSet.Contains(targetUserId)) // O(1)
{
    // ...
}
```

---
## 八、查詢效能原理：為什麼是 O(1)？

### 8.1 雜湊運作原理（像置物櫃系統）

HashSet\<T> 就像是一個有編號的「置物櫃系統」。

#### 存入資料的過程

當你把一個物件放入 HashSet 時，.NET 內部會做以下動作：

```
步驟 1: 呼叫該物件的 GetHashCode() 方法，算出一組數字
        例如：User { Id = 1 } → GetHashCode() → 1024

步驟 2: 透過演算法將這組數字對應到內部的某個「桶子 (Bucket)」索引位置
        例如：1024 % 陣列大小 16 = 8 → 放到 8 號桶子

步驟 3: 將資料直接丟進那個桶子裡
```

---
#### 查詢資料的過程

當你呼叫 `HashSet.Contains(target)` 時：

```
步驟 1: 系統不需要從頭開始找

步驟 2: 它直接計算 target 的 GetHashCode()
        例如：target { Id = 1 } → GetHashCode() → 1024

步驟 3: 它直接「跳」到對應的桶子去看「有沒有東西」
        例如：1024 % 16 = 8 → 直接檢查 8 號桶子

步驟 4: 如果有，再用 Equals() 確認一下是不是同一個物件
```

---
#### 為什麼快？

這個過程就像你拿著鑰匙去開置物櫃：

- 你知道號碼是 **5 號**，直接走去 **5 號櫃**打開就好
- **不用**打開 1, 2, 3, 4 號櫃子檢查
- 所以無論櫃子有 **10 個**還是 **100 萬個**，你花的時間幾乎是一樣的

這就是為什麼 HashSet 的查詢效能是 **O(1)**。

---
### 8.2 視覺化比較

```
List<T> 的 Contains (O(n)):
櫃子: [1] [2] [3] [4] [5] [6] [7] [8] [9] [10]
查詢 5: 
    檢查 [1] → 不是
    檢查 [2] → 不是
    檢查 [3] → 不是
    檢查 [4] → 不是
    檢查 [5] → 找到了！✓
    
平均要檢查一半的資料，資料越多越慢

---

HashSet<T> 的 Contains (O(1)):
查詢 5:
    計算 HashCode → 跳到 5 號桶子
    檢查 [5] → 找到了！✓
    
不管有多少資料，都是直接跳到目標
```

---
### 8.3 注意事項：雜湊碰撞 (Hash Collision)

雖然說是 O(1)，但如果有發生**雜湊碰撞**（即兩個不同的物件算出一樣的 HashCode）：

```csharp
User user1 = new User { Id = 1 }; // HashCode = 1024
User user2 = new User { Id = 17 }; // HashCode 剛好也是 1024
```

這時候兩個物件會被放到同一個桶子裡：

```
8 號桶子: [user1] → [user2] (形成鏈結串列)
```

當查詢時，HashSet 會：

1. 跳到 8 號桶子
2. 發現裡面有多個物件
3. 用 `Equals()` 逐一比對

這時效率會稍微下降，但在良好的雜湊演算法下：

- 碰撞機率很低
- 即使碰撞，每個桶子裡的物件數量也很少
- 平均仍視為 **O(1)**

---
## 九、常見陷阱與注意事項

### 9.1 不要在 foreach 中修改集合

```csharp
// ❌ 錯誤：會拋出 InvalidOperationException
var users = new HashSet<User> { user1, user2, user3 };

foreach (var user in users)
{
    if (user.Id == 1)
    {
        users.Remove(user); // ❌ 錯誤：在走訪時修改集合
    }
}

// ✓ 正確：先收集要移除的項目，再統一移除
var toRemove = new List<User>();

foreach (var user in users)
{
    if (user.Id == 1)
    {
        toRemove.Add(user);
    }
}

foreach (var user in toRemove)
{
    users.Remove(user);
}
```

---
### 9.2 不保證順序

```csharp
var numbers = new HashSet<int> { 3, 1, 4, 1, 5, 9, 2, 6 };

foreach (var num in numbers)
{
    Console.Write(num + " ");
}
// 輸出可能是: 1 2 3 4 5 6 9
// 也可能是: 3 1 4 5 9 2 6
// 順序不確定！
```

如果需要「不重複」且「保持加入順序」：

- 在較舊的 .NET Framework 可能看似有序，但不可依賴此行為
- 考慮使用 `List<T>` + `Distinct()` 或自行實作

---
### 9.3 GetHashCode 實作很重要

```csharp
// ❌ 糟糕的實作
public override int GetHashCode()
{
    return 1; // 所有物件都回傳相同的 HashCode
}

// 結果：
// - 所有物件都會被丟到同一個桶子
// - HashSet 退化成鏈結串列
// - 效能從 O(1) 變成 O(n)
// - 完全失去使用 HashSet 的意義

// ✓ 正確的實作
public override int GetHashCode()
{
    return this.Id.GetHashCode(); // 根據實際用來比較的屬性
}
```

---
### 9.4 多個屬性的 GetHashCode 實作

如果你需要根據多個屬性來判斷重複（例如：ID + 部門代號）：

```csharp
public class Employee : IEquatable<Employee>
{
    public int Id { get; set; }
    public string Department { get; set; }
    
    public bool Equals(Employee other)
    {
        if (other is null) return false;
        // 比較 Id 與 Department 是否皆相同
        return this.Id == other.Id && 
               this.Department == other.Department;
    }
    
    // 多個屬性的 HashCode 計算
    public override int GetHashCode()
    {
        // ------------------------------------------------------------
		// [現代化寫法]：.NET Core 2.1+ 推薦寫法
		// ------------------------------------------------------------
		// 優點 1：自動處理 Null 安全
		//        它會自動判斷 Department 是否為 null，不用手寫 ?. 判斷。
		//
		// 優點 2：演算法優化 (xxHash32)
		//        底層使用更先進的混淆演算法，比舊式乘法更能避免雜湊碰撞。
		//
		// 優點 3：參數順序敏感
		//        Combine(A, B) 與 Combine(B, A) 會產生不同的結果，這在邏輯上更嚴謹。
		//
		// 注意事項：
		//        若要在您的 GTS (.NET Framework 4.8) 系統使用此寫法，
		//        必須從 NuGet 安裝套件：Microsoft.Bcl.HashCode
		return HashCode.Combine(Id, Department);		
		
		// ------------------------------------------------------------
		// [傳統寫法]：舊版 .NET Framework 常見 (不推薦新專案使用)
		// ------------------------------------------------------------
		// unchecked // 避免溢位檢查，稍微提升效能
		// {
		//      // 這些 17, 23 是質數 (Magic Numbers)，用來減少碰撞，但可讀性差
		//      int hash = 17;
		//      
		//      hash = hash * 23 + Id.GetHashCode();
		//
		//      // 缺點：必須手動處理 Null，否則 Department 為 null 時會拋出異常
		//      hash = hash * 23 + (Department?.GetHashCode() ?? 0);
		//      
		//      return hash;
		// }        
    }
    
    public override bool Equals(object obj) => Equals(obj as Employee);
}
```

---
## 十、總結

### 10.1 何時使用 HashSet

✓ **適合使用 HashSet 的場景**：

- 需要儲存**唯一值**（不允許重複）
- 需要**大量資料比對**（頻繁使用 `Contains`）
- **不需要順序**或索引存取
- 需要進行**集合運算**（聯集、交集、差集）

✓ **適合使用 List 的場景**：

- 需要**索引存取**（`list[0]`, `list[1]`）
- 需要**保持順序**
- **允許重複**資料
- 需要頻繁的**插入/移除**特定位置的元素

---
### 10.2 使用自定義物件的檢查清單

當你在 HashSet 中使用自定義類別時，請確保：

- [ ] 實作 `IEquatable<T>` 介面
- [ ] 實作 `Equals(T other)` 方法（VIP 通道）
- [ ] 覆寫 `GetHashCode()` 方法（分類依據）
- [ ] 覆寫 `Equals(object obj)` 方法（相容性）
- [ ] 確保 `Equals` 為 true 的物件，`GetHashCode` 回傳相同值
- [ ] `GetHashCode` 的實作要根據實際用來比較的屬性

---
### 10.3 效能總結

|操作|List<T>|HashSet<T>|
|---|---|---|
|Add|O(1)|O(1)|
|Remove|O(n)|O(1)|
|Contains|**O(n)**|**O(1)** ⭐|
|順序|保證|不保證|
|重複|允許|不允許|

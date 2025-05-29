---
title: 'C# 的 stack 堆疊、heap 堆積'
tags: ['C# 程式']

---

我很樂意為您介紹 C# 程式設計中的 Stack (堆疊) 和 Heap (堆積)。這是理解 C# 記憶體管理的重要概念。

## Stack (堆疊)

Stack 是一種具有後進先出 (LIFO) 特性的記憶體區域，主要用於：

- 儲存值類型變數 (Value Types)：如 int、char、bool、struct 等
- 儲存物件的參考 (但不是物件本身)
- 追蹤程式執行的方法呼叫
- 儲存方法中的局部變數

特點：
- 大小固定，通常較小 (預設約 1MB)
- 自動管理記憶體配置與釋放
- 存取速度快
- 當方法結束時，其在 Stack 上的記憶體自動釋放

## Heap (堆積)

Heap 是一種結構較不嚴謹的記憶體區域，主要用於：

- 儲存參考類型 (Reference Types) 的實際資料，如 class、array、string 等
- 可能會隨著程式執行而增長或縮小的資料

特點：
- 大小較大，理論上僅受系統記憶體限制
- 由 .NET 的垃圾回收器 (Garbage Collector) 管理
- 存取速度較 Stack 慢
- 資料不會在方法結束時自動釋放，而是由 GC 決定何時清理

## 例子說明

```csharp
public void ExampleMethod()
{
    // Stack 上的值類型
    int x = 10;         // x 整個存在 Stack 上
    bool isValid = true; // isValid 整個存在 Stack 上
    
    // Heap 上的參考類型
    string name = "張三";   // name 的參考存在 Stack 上，"張三" 字串內容存在 Heap 上
    
    // 類別實例
    Person person = new Person(); // person 的參考存在 Stack 上，Person 物件存在 Heap 上
    
    // 值類型的陣列
    int[] numbers = new int[5]; // numbers 的參考存在 Stack 上，陣列本身和內容存在 Heap 上
    
    // 結構
    MyStruct myStruct = new MyStruct(); // 整個 myStruct 都存在 Stack 上
}

// 自定義結構
struct MyStruct
{
    public int Id;
    public string Name; // Name 是參考，存在 Stack 上，但它指向的字串在 Heap 上
}

// 自定義類別
class Person
{
    public int Age;     // Age 存在 Heap 上 (因為 Person 是類別，位於 Heap)
    public string Name; // Name 的參考和 Name 指向的字串內容都在 Heap 上
}
```

## 記憶體回收機制

- Stack 的記憶體：當方法執行完畢，Stack 幀 (Stack Frame) 被彈出，相關記憶體自動釋放
- Heap 的記憶體：由 Garbage Collector 定期執行回收，回收沒有參考的物件

## 值類型與參考類型的區別

### 值類型 (Stack)
- struct、enum、基本數據類型 (int、float、bool 等)
- 直接包含資料
- 作為參數傳遞時是複製值

### 參考類型 (Heap)
- class、interface、delegate、array、string
- 包含指向 Heap 上資料的參考
- 作為參數傳遞時是複製參考 (但兩個參考指向同一個對象)

了解 Stack 和 Heap 的區別對於寫出高效能的 C# 程式非常重要，尤其是在處理大量資料或記憶體密集型應用程式時。
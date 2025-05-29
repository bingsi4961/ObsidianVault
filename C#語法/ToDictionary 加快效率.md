---
title: ToDictionary 加快效率
tags: ['C# 程式', LINQ]

---

讓我用一個具體的例子來解釋這個概念。

想像你有一本通訊錄和一張便利貼：
- 通訊錄是按照姓名排序的列表（類似 FirstOrDefault 的情況）
- 便利貼是一個快速索引表（類似 Dictionary）

**使用 FirstOrDefault（通訊錄）的情況：**
```csharp
// 假設有 100 個聯絡人
var contacts = new List<Contact>();  // 通訊錄
var targetContact = contacts.FirstOrDefault(c => c.Name == "John");  // 找 John
```
當你要找 "John" 時，你需要：
1. 從第一頁開始翻
2. 一頁一頁往後找
3. 直到找到 "John" 或翻完整本通訊錄

這就是 O(n) 的時間複雜度，n 是聯絡人數量。如果有 100 個聯絡人，最糟的情況下需要翻 100 次。

**使用 Dictionary（便利貼索引）的情況：**
```csharp
// 建立快速索引
var contactDict = contacts.ToDictionary(c => c.Name);  // 便利貼索引
var targetContact = contactDict["John"];  // 直接查找 John
```
這就像：
1. 先花一點時間製作索引（便利貼）
2. 之後要找人時，直接看索引就能找到目標頁碼
3. 直接翻到那一頁

這是 O(1) 的時間複雜度，不管有幾個聯絡人，都只需要一次查找。

**效能比較：**
```csharp
// 原本的程式碼（較慢）
foreach (var vmDesc in relatedDescs)
{
    // 每次都要搜尋整個集合
    var entyDesc = updDescs4DB.FirstOrDefault(
        dbDesc => dbDesc.KeypartId == vmKp.Id && 
                 dbDesc.KpcolumnId == vmDesc.KPColumnId);
    // ...
}

// 優化後的程式碼（較快）
// 先建立索引
var descriptionDict = updDescs4DB
    .Where(dbDesc => dbDesc.KeypartId == vmKp.Id)
    .ToDictionary(dbDesc => dbDesc.KpcolumnId);

foreach (var vmDesc in relatedDescs)
{
    // 直接用索引查找，速度快很多
    if (descriptionDict.TryGetValue(vmDesc.KPColumnId, out var entyDesc))
    {
        // ...
    }
}
```

這種優化在處理大量資料時特別明顯：
- 如果有 1,000 筆資料，FirstOrDefault 可能需要執行 1,000 × 1,000 = 1,000,000 次比較
- 使用 Dictionary 則只需要 1,000 次（建立索引）+ 1,000 次（查找），總共 2,000 次操作

雖然建立 Dictionary 需要一些初始成本，但只要後續查找次數夠多，這個投資就非常值得。就像製作通訊錄索引雖然需要時間，但能大幅加快之後的查找速度。
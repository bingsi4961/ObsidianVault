---
date: 2025-07-15 15:15
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
#### 📑 [[歡迎]]
#### 📑 [[歡迎]]

---
## RouteValueDictionary 基本概念

`RouteValueDictionary` 是一個特殊的字典類別，專門用來存放路由參數。它可以：

1. 接受匿名物件並自動轉換成鍵值對
2. 像字典一樣使用索引器新增或修改值

## 步驟解析

```csharp
return RedirectToAction("Detail", new RouteValueDictionary(condition)
{
    ["groupid"] = groupid,
    ["stageid"] = stageid,
    ["deviceid"] = deviceid,
    ["skuId"] = skuId
});
```

### 步驟 1：建立 RouteValueDictionary

```csharp
new RouteValueDictionary(condition)
```

- 這會自動將 `condition` 物件的所有公開屬性轉換成字典
- 假設 `condition` 有屬性 `Name`、`Status`、`Date`，就會自動產生對應的鍵值對

### 步驟 2：使用集合初始化語法新增額外參數

```csharp
{
    ["groupid"] = groupid,
    ["stageid"] = stageid,
    ["deviceid"] = deviceid,
    ["skuId"] = skuId
}
```

- 使用 `[鍵] = 值` 的語法新增額外的路由參數
- 這些會與 `condition` 的屬性合併

## 完整範例

假設您有這樣的 `condition` 物件：

```csharp
public class SearchCondition
{
    public string Name { get; set; }
    public string Status { get; set; }
    public DateTime? StartDate { get; set; }
    public int? CategoryId { get; set; }
}

// 在 Controller 中使用
var condition = new SearchCondition
{
    Name = "測試產品",
    Status = "Active",
    StartDate = DateTime.Now,
    CategoryId = 1
};

int groupid = 100;
int stageid = 200;
int deviceid = 300;
int skuId = 400;
```

使用：

```csharp
return RedirectToAction("Detail", new RouteValueDictionary(condition)
{
    ["groupid"] = groupid,
    ["stageid"] = stageid,
    ["deviceid"] = deviceid,
    ["skuId"] = skuId
});
```

## 最終產生的 URL 會是：

```
/Controller/Detail?Name=測試產品&Status=Active&StartDate=2024-01-01&CategoryId=1&groupid=100&stageid=200&deviceid=300&skuId=400
```

## 注意事項

1. **屬性名稱會保持原樣**：`condition` 物件的屬性名稱會直接當作參數名稱
2. **null 值會被忽略**：如果 `condition` 的某個屬性是 `null`，該參數不會出現在 URL 中
3. **覆蓋機制**：如果 `condition` 有與額外參數同名的屬性，額外參數會覆蓋原有值

```csharp
// 如果 condition 也有 groupid 屬性，下面的 groupid 會覆蓋它
return RedirectToAction("Detail", new RouteValueDictionary(condition)
{
    ["groupid"] = groupid  // 這個值會覆蓋 condition.groupid
});
```

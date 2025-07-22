---
date: 2025-07-15 17:31
aliases: 
tags:
  - NetCore_MVC
  - MVC5
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

```CSharp
using System.ComponentModel.DataAnnotations;
using System.Reflection;
using System.Linq.Expressions;

public static class DisplayNameHelper
{
    public static string GetDisplayName<T>(Expression<Func<T, object>> expression)
    {
        var memberExpression = GetMemberExpression(expression);
        if (memberExpression?.Member is PropertyInfo property)
        {
            var displayAttribute = property.GetCustomAttribute<DisplayAttribute>();
            return displayAttribute?.Name ?? 
                   displayAttribute?.GetName() ?? 
                   property.Name;
        }
        return string.Empty;
    }
    
    private static MemberExpression GetMemberExpression<T>(Expression<Func<T, object>> expression)
    {
        if (expression.Body is MemberExpression memberExpression)
            return memberExpression;
            
        if (expression.Body is UnaryExpression unaryExpression && 
            unaryExpression.Operand is MemberExpression operand)
            return operand;
            
        return null;
    }
}

// 使用方式
public IActionResult DetailExport(int groupid, int stageid, int deviceid, int skuId, [FromQuery] ReadSkuTableCondition condition)
{
    // 假設您的 Item 類型是 ItemViewModel
    string displayName = DisplayNameHelper.GetDisplayName<ItemViewModel>(x => x.PpmDesc);
    
    workSheet.Cell(1, 1).Value = displayName;
}
```
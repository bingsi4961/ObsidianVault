---
date: 2025-09-12 15:34
aliases:
tags:
  - EntityFramework_6
  - EntityFramework_Core
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

```csharp
/// <summary>
/// 取得 Stage 與 SKU 資料
/// </summary>        
public CommonResponse<List<StageSettingVM>> GetSkuAndStage(int groupId, int deviceCodeId)
{
    var condition = ExpressionBuilder.True<StageSetting>();
    condition = condition.And(s => s.GroupId == groupId);
    condition = condition.And(s => s.IsValid.Value);

    // 在 join 階段加上 where 條件，減少處理的資料量
    // 在資料一對多的情況下，join 是會組成多筆資料
    var rawData = (
        from stage in this._db.StageSetting.AsNoTracking().Where(condition)
        join skus in this._db.Skusetting.Where(skus => skus.IsValid.Value)
            on stage.Id equals skus.StageId
        join skut in this._db.SkuTable.Where(skut => skut.DeviceCodeId == deviceCodeId)
            on skus.Id equals skut.SkuId
        join item in this._db.SkuTableItem
            on skut.Id equals item.SkuTableId
        select new
        {
            StageId = stage.Id,
            StageName = stage.Name,
            StageSequence = stage.Sequence,
            SkusId = skus.Id,
            SkusName = skus.Name,
            SkusSequence = skus.Sequence
        })
        .Distinct()
        .ToList();   // 先執行 SQL 語法，將資料送回記憶體

    var data = rawData
        .OrderBy(x => x.StageSequence)
            .ThenBy(x => x.SkusSequence)
        .GroupBy(x => new { x.StageId, x.StageName })
        .Select(stageGroup => new StageSettingVM
        {
            Id = stageGroup.Key.StageId,
            Name = stageGroup.Key.StageName,
            SKUSettingVMs = stageGroup
                .Select(skus => new SKUSettingVM
                {
                    Id = skus.SkusId,
                    Name = skus.SkusName
                }).ToList()

        }).ToList();

    var resp = new CommonResponse<List<StageSettingVM>>();
    resp.Status = CommonResponse.StatusEnum.Success;
    resp.Data = data;
    return resp;
}
```

1. Include() 方法會自動根據導航屬性，和外鍵關係來執行 JOIN，不需要手動指定 JOIN 條件。

2. <mark style="background: #FFF3A3A6;">但是 LINQ Query Syntax（查詢語法）中的 join，必須手動指定 equals 條件</mark>。
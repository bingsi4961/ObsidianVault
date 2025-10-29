---
date: 2025-10-29 15:33
aliases:
tags:
  - jQuery
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

```html
<input type="button" id="batchEdit" value="批次編輯" class="btn btn-primary" />

@foreach (var item in Model.Data)
{
	<input type="checkbox" value="@item.GroupId" class="cbx-batch-edit" />
}
```

```javascript
// 全選 / 取消全選
$('#allCbxBatchEdit').on('change', function() {
    $('.cbx-batch-edit').prop('checked', this.checked);
});

// 單一勾選框變更時，更新全選框狀態
$('.cbx-batch-edit').on('change', () => {
    const $items = $('.cbx-batch-edit');
    $('#allCbxBatchEdit').prop('checked', $items.length === $items.filter(':checked').length);
});
```

---
title: 動態建立 Form 表單及 Submit
tags: [jQuery]

---

```javascript
$('#btExport').on('click', function() {
    var checkedValues = $('input[name=checkGroup]:checked').map(function() {
        return $(this).val();
    }).toArray();
    
    // 使用 POST 請求
    $('<form>')
        .attr('method', 'post')
        .attr('action', '@Url.Action("ExportMemberList")')
        .append(
            $('<input>')
                .attr('type', 'hidden')
                .attr('name', 'check')
                .val(checkedValues)
        )
        .appendTo('body')
        .submit()
        .remove();
});
```
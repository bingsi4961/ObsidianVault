---
title: 將所有的 input type="text" 的輸入值進行 Trim() 方法
tags: [jQuery, 技術文件]

---

```javascript=
$("form").submit(function() {
    $("input[type='text']").each(function() {
        $(this).val($.trim($(this).val()));
    });
});


$(function () {    
    // 設定網頁上所有的 input type='text' 發生 onBlur 時，進行 Trim()
    $("input[type='text']").blur(function () {
        $(this).val($.trim($(this).val()));
    });
});
```
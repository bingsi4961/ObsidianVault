---
date: 2025-06-24 10:25
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

```javascript
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
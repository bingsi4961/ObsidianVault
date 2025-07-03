---
date: 2025-07-03 10:59
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

```javascript=
$(document).on('input','input[name="UnitConsumption"]',function(){
    this.value = this.value.replace(/[^0-9]/g,'').replace(/^0/,'');
});
```

---
title: input text 欄位只允許輸入正整數
tags: [jQuery, JavaScript]

---

```javascript=
$(document).on('input','input[name="UnitConsumption"]',function(){
    this.value = this.value.replace(/[^0-9]/g,'').replace(/^0/,'');
});
```

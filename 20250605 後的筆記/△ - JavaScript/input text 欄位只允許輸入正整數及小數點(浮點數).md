---
date: 2025-10-29 16:29
aliases:
tags:
  - JavaScript
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

```javascript=
$(document).on('input','input[name="UnitConsumption"]',function() {
    this.value = this.value
	    // 1. 移除所有 "非數字" 和 "非小數點" 的字元
	    .replace(/[^0-9\.]/g, '')
        
	    // 2. 移除多餘的小數點 (只保留第一個)
	    //    ( \..* ) 會捕獲第一個小數點及之後的所有內容
	    //    \.     會匹配 "之後" 出現的任何小數點
	    //    $1     代表只保留第一個捕獲組 (即第一個小數點和它後面的內容)
	    .replace(/(\..*)\./g, '$1')
	        
	    // 3. 處理前導零 (例如 05 -> 5, 05.1 -> 5.1, 但 0.5 和 0 不變)
	    //    ^0    匹配開頭的 0
	    //    (\d)  匹配 0 後面的 "數字" (不包含小數點)
	    .replace(/^0(\d)/, '$1')
	
	    // 4. 如果開頭是小數點,自動補上 0 (例如 .51 -> 0.51)
	    //    ^\.   匹配開頭的小數點
	    .replace(/^\./, '0.');
});
```

---
date: 2026-01-04 14:35
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

```javascript
/**
 * 字串正規化函式
 * 處理 null/undefined/NaN 值，並可選擇性地執行 trim 和 toLowerCase
 *  
 * @example
 * // 基本用法（預設只做 trim）
 * normalize('  Hello  ')              // 'Hello'
 * normalize(null)                     // ''
 * normalize(undefined)                // ''
 * normalize(NaN)                      // ''
 * normalize(123)                      // '123'
 * 
 * @example
 * // trim + toLowerCase
 * normalize('  HELLO  ', { lower: true })  // 'hello'
 * 
 * @example
 * // 只做 toLowerCase，不做 trim
 * normalize('  HELLO  ', { trim: false, lower: true })  // '  hello  '
 * 
 * @example
 * // 都不做，只處理 null/undefined/NaN
 * normalize('  HELLO  ', { trim: false })  // '  HELLO  '
 */
function normalize(value, options = {}) {
    // 處理 null、undefined、NaN，統一轉為空字串
    var result = (value == null || Number.isNaN(value)) ? '' : value.toString();
    
    // trim 預設開啟（除非明確設定 trim: false）
    if (options.trim !== false) {
        result = result.trim();
    }
    
    // toLowerCase 預設關閉（需明確設定 lower: true）
    if (options.lower) {
        result = result.toLowerCase();
    }
    
    return result;
}
```

---

## 快速參考表

|呼叫方式|trim|lower|說明|
|---|:-:|:-:|---|
|`normalize(value)`|✓|✗|預設：只做 trim|
|`normalize(value, { lower: true })`|✓|✓|trim + 轉小寫|
|`normalize(value, { trim: false, lower: true })`|✗|✓|只轉小寫|
|`normalize(value, { trim: false })`|✗|✗|只處理 null/NaN|

---

## 設計重點

1. **安全處理空值**：`value == null` 同時涵蓋 `null` 和 `undefined`
2. **預設行為合理**：trim 預設開啟符合大多數使用情境
3. **選項設計巧妙**：用 `!== false` 判斷讓 `undefined` 也視為啟用
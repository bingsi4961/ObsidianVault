---
date : 2025-10-29 16:29
aliases:
  - 別名測試1
  - 別名測試2
tags:
  - 標籤測試1
  - 標籤測試2

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

我幫你重新排版這份筆記：

---
## 1. 轉換函式 (Parsing vs. Converting)

### `parseInt(value)` - 只取整數

**工作：** 從字串開頭解析「整數」

**規則：**

1. 會先將 `value` 轉為字串（例如 `null` 變 `"null"`）
2. 忽略「開頭」空白
3. 從頭開始讀取，直到遇到第一個非數字字元（包含小數點 `.`）就停止
4. 如果第一個非空白字元就不是數字，回傳 `NaN`

**範例：**

```javascript
parseInt("123.45")    // → 123 (遇到 . 停止)
parseInt("123abc")    // → 123 (遇到 a 停止)
parseInt("abc123")    // → NaN (開頭 a 不是數字)
parseInt(null)        // → parseInt("null") → NaN
```

```javascript
// 從左到右解析，直到遇到非數字字元就停止
// 只返回整數部分，不會四捨五入
parseInt(undefined)        // NaN
parseInt(null)             // NaN
parseInt("")               // NaN
parseInt(" ")              // NaN
parseInt(true)		       // NaN
parseInt("0")              // 0
parseInt("123.456")        // 123
parseInt("  123.456  ")    // 123
parseInt("123.456abc")     // 123
parseInt("abc123")         // NaN
```

---
### `parseFloat(value)` - 可取小數

**工作：** 從字串開頭解析「浮點數（小數）」

**規則：** 與 `parseInt` 幾乎相同，但它會接受「第一個」小數點 `.`

**範例：**

```javascript
parseFloat("123.45")     // → 123.45
parseFloat("123.45abc")  // → 123.45 (遇到 a 停止)
parseFloat("abc123.45")  // → NaN (開頭 a 不是數字)
```

```javascript
// 從左到右解析，直到遇到非數字字元就停止
// 可處理小數
parseFloat(undefined)        // NaN
parseFloat(null              // NaN
parseFloat("")               // NaN
parseFloat(" ")              // NaN
parseFloat(true)             // NaN
parseFloat("0")              // 0
parseFloat("123.456")        // 123.456
parseFloat("  123.456  ")    // 123.456
parseFloat("123.456abc")     // 123.456
parseFloat("abc123.456")     // NaN
```

---
### `Number(value)` - 嚴格轉換

**工作：** 嘗試將 **「整個值」** 轉換為數字

**規則：**

1. 不只是解析開頭，它要求整個值（修剪掉前後空白後）都必須是合法的數字
2. 對 `null`、`""`、`true` 有特殊轉換規則

**範例：**

```javascript
Number("123.45")      // → 123.45
Number("123.45abc")   // → NaN (與 parseFloat 不同，因為 "abc" 不是數字)
Number(null)          // → 0
Number("")            // → 0
Number(" ")           // → 0 (修剪空白後變 "")
Number(true)          // → 1
```

```javascript
// 最嚴格的轉換
Number(undefined)     // NaN
Number(null)          // 0
Number("")            // 0
Number("  ")          // 0
Number(true)          // 1
Number("0")           // 0
Number("123.456")     // 123.456
Number("  123.456  ") // 123.456
Number("123.456abc")  // NaN
Number("abc123.456")  // NaN
```

---
## 2. `NaN` 檢查函式

### `isNaN(value)` - (舊版，會轉型)

**工作：** 「如果我嘗試把 `value` 轉成數字，結果是不是 `NaN`？」

**規則：** 它會先對 `value` 執行 `Number(value)`，然後才檢查結果

**範例：**

```javascript
isNaN("abc")       // → true (因為 Number("abc") 是 NaN)
isNaN(undefined)   // → true (因為 Number(undefined) 是 NaN)
isNaN(null)        // → false (因為 Number(null) 是 0)
isNaN("")          // → false (因為 Number("") 是 0)
```

```javascript
isNaN(undefined)     // true
isNaN(null)          // false
isNaN("")            // false
isNaN("  ")          // false
isNaN(true)          // false
isNaN("0")           // false
isNaN("123.456")     // false
isNaN("  123.456  ") // false
isNaN("123.456abc")  // true
isNaN("abc123.456")  // true
```

---

### `Number.isNaN(value)` - (新版，嚴格檢查)

**工作：** 「`value` 本身是不是那個特殊值 `NaN`？」

**規則：** <mark style="background: #FFF3A3A6;">絕對不會做型別轉換 ( Number(x) )</mark>。它只在 `value` 已經是 `NaN` 時才回傳 `true`

**主要用途：** 用來取代 `NaN === NaN`（會回傳 `false`）的怪異行為

**範例：**

```javascript
Number.isNaN("abc")   // → false (因為 "abc" 是一個 string，不是 NaN)
Number.isNaN(null)    // → false (因為 null 不是 NaN)
Number.isNaN(NaN)     // → true

let result = 0 / 0;
Number.isNaN(result); // → true (正確檢查運算失敗)
```

---

## 快速對照表

|函式|類型|會轉型？|適用情境|
|---|---|---|---|
|`parseInt()`|轉換|是|從字串開頭解析整數|
|`parseFloat()`|轉換|是|從字串開頭解析浮點數|
|`Number()`|轉換|是|嚴格轉換整個值|
|`isNaN()`|檢查|是|檢查能否轉為數字（不推薦）|
|`Number.isNaN()`|檢查|否|嚴格檢查是否為 NaN（推薦）|
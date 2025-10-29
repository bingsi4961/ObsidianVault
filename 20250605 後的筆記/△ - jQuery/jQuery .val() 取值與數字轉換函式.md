---
date: 2025-10-29 17:25
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
#### 📑 [[parseInt、parseFloat、Number、isNaN 和 Number.isNaN 的核心差異]]

---
## 核心觀念

### `.val()` 的回傳規則

jQuery 的 `$('#txtTest').val()` **只會回傳兩種結果：**

1. **`string` (字串)**：只要元件存在，`val()` 就會回傳字串
2. **`undefined`**：當 `$('#txtTest')` 選取不到任何元件時

**重要：**

- `val()` 不會回傳 `null`、`true` 或 `false`
- HTML 屬性中的 `value=true` 或 `value=null` 都會被讀取為字串 `"true"` 和 `"null"`

---
## 第一部分：各種情境下 `txtTestVal` 的結果

```javascript
const txtTestVal = $('#txtTest').val();
```

|情境|HTML 範例|`txtTestVal` 結果|
|---|---|---|
|1. 沒有 txtTest 元件|`(元件不存在)`|`undefined`|
|2. value=null|`<input value=null>`|`"null"` (字串)|
|2. value="null"|`<input value="null">`|`"null"` (字串)|
|3. value=""|`<input value="">`|`""` (空字串)|
|3. value=|`<input value=>`|`""` (空字串)|
|3. 沒有設定 value|`<input>`|`""` (空字串)|
|4. value=" "|`<input value=" ">`|`" "` (包含空白的字串)|
|5. value="true"|`<input value="true">`|`"true"` (字串)|
|5. value=true|`<input value=true>`|`"true"` (字串)|
|6. value="false"|`<input value="false">`|`"false"` (字串)|
|6. value=false|`<input value=false>`|`"false"` (字串)|
|7. value="0"|`<input value="0">`|`"0"` (字串)|
|-|`<input value=" 123.456 ">`|`" 123.456 "` (字串，保留空白)|

---
## 第二部分：套用數字轉換函式的完整結果

將上述 `txtTestVal` 的結果套用到 5 個函式中：

|txtTestVal (輸入)|`parseInt(val)`|`parseFloat(val)`|`Number(val)`|`isNaN(val)`|`Number.isNaN(val)`|
|---|---|---|---|---|---|
|`undefined` (元件不存在)|`NaN`|`NaN`|`NaN`|`true`|`false`|
|`"null"`|`NaN` (n 非數字)|`NaN` (n 非數字)|`NaN`|`true`|`false`|
|`""` (空字串)|`NaN`|`NaN`|`0`|`false`|`false`|
|`" "` (空白字串)|`NaN`|`NaN`|`0`|`false`|`false`|
|`"true"`|`NaN` (t 非數字)|`NaN` (t 非數字)|`NaN`|`true`|`false`|
|`"false"`|`NaN` (f 非數字)|`NaN` (f 非數字)|`NaN`|`true`|`false`|
|`"0"`|`0`|`0`|`0`|`false`|`false`|
|`" 123.456 "`|`123`|`123.456`|`123.456`|`false`|`false`|

---
## 🚨 實務開發的重要陷阱

### 陷阱 1：`"null"` (字串) vs. `null` (空值)

```javascript
Number(null)      // → 0
Number("null")    // → NaN
```

**問題：** 因為 `.val()` 拿到的是 `"null"` 字串，所以在 `Number()` 轉換時會失敗（變 `NaN`），這與 `Number(null)` 的行為完全不同。

---
### 陷阱 2：`"true"` (字串) vs. `true` (布林值)

```javascript
Number(true)      // → 1
Number("true")    // → NaN
```

**問題：** 同樣，`.val()` 拿到的字串 `"true"` 在轉換時會失敗。

---
### 陷阱 3：`""` 和 `" "` (空字串/空白) 的陷阱 ⚠️ **最危險**

```javascript
Number("")        // → 0
Number(" ")       // → 0
isNaN("")         // → false
isNaN(" ")        // → false
```

**問題：** 如果用 `isNaN($('#txtTest').val())` 來檢查使用者是否「沒有輸入數字」：

- 當使用者沒填 (`""`) 或只填空白 (`" "`) 時
- `isNaN` 會回傳 `false`
- 程式會誤以為 `0` 是一個有效的輸入
- **這通常不是你想要的結果！**

---

## ✅ 實務建議：正確的驗證方式

### 方案 1：先 trim 再驗證

```javascript
const txtTestVal = $('#txtTest').val().trim();

// 檢查是否為空
if (txtTestVal === '') {
    alert('請輸入數字');
    return;
}

// 檢查是否為有效數字
const numValue = Number(txtTestVal);
if (isNaN(numValue) || numValue <= 0) {
    alert('請輸入有效的正數');
    return;
}
```

### 方案 2：組合驗證（最嚴謹）

```javascript
const txtTestVal = $('#txtTest').val().trim();
const numValue = Number(txtTestVal);

// 檢查：不是空值、是數字、大於 0
if (txtTestVal && !isNaN(numValue) && numValue > 0) {
    // 有效的數字
    selectedGroupIds.push(txtTestVal);
} else {
    alert('請輸入有效的正數');
}
```

### 方案 3：使用正規表達式

```javascript
const txtTestVal = $('#txtTest').val().trim();

// 只接受正整數或正小數
if (/^\d+(\.\d+)?$/.test(txtTestVal) && Number(txtTestVal) > 0) {
    selectedGroupIds.push(txtTestVal);
} else {
    alert('請輸入有效的正數');
}
```

---
## 📋 驗證流程圖

```
使用者輸入
    ↓
取得 .val()
    ↓
.trim() 移除前後空白
    ↓
檢查是否為空字串 ("")
    ├─ 是 → 提示錯誤
    └─ 否 ↓
轉換為數字 Number()
    ↓
檢查 isNaN() 和範圍
    ├─ 是 NaN 或 ≤ 0 → 提示錯誤
    └─ 否 → 加入陣列
```

---
## 🎯 重點總結

1. **`.val()` 永遠回傳字串或 `undefined`**，不會回傳 `null` 或布林值
2. <mark style="background: #FFF3A3A6;">空字串和空白字串 在 Number() 和 isNaN() 中會被轉為 0（陷阱！）</mark>
3. **HTML 屬性的 `value=null`** 會被讀取為字串 `"null"`
4. **驗證數字前必須先 `.trim()`** 移除空白
5. **推薦使用組合驗證**：`txtTestVal && !isNaN(numValue) && numValue > 0`
6. **`Number.isNaN()` 比 `isNaN()` 更嚴格**，但不適合用於驗證使用者輸入
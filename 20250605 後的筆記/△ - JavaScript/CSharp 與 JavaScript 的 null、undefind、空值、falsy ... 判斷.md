---
date: 2025-11-26 16:52
aliases:
tags:
  - JavaScript
  - CSharp_語法
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
## 核心概念

C# 的 `??` 是**空值聯合運算符**，專門處理 `null` 值。JavaScript 的 `||` 則處理所有**假值（falsy values）**，範圍更廣。

---
## JavaScript 的假值（Falsy Values）

```javascript
false, null, undefined, 0, "", NaN
```

---
## 運算符比較表

|   運算符   |     語言      | 檢查範圍                | 說明                                |
| :-----: | :---------: | ------------------- | --------------------------------- |
|   ??    |     C#      | `null`              | 只在左側為 `null` 時返回右側值               |
|   ??    | JS (ES2020) | `null`, `undefined` | 只在左側為 `null` 或 `undefined` 時返回右側值 |
|  \|\|   |     JS      | 所有 falsy 值          | 左側為任何假值時返回右側值                     |
| != null |     JS      | `null`, `undefined` | 同時檢查這兩者                           |
|   !變數   |     JS      | 所有 falsy 值          | 會檢查變數是否為 **falsy 值**              |

---
## 行為差異對照表（含範例）

| 左側值         | C# ??                          | JS \|\|      | JS ??        |
| ----------- | ------------------------------ | ------------ | ------------ |
| `null`      | 返回右側                           | 返回右側         | 返回右側         |
| `undefined` | (不適用)                          | 返回右側         | 返回右側         |
| `""` 空字串    | 返回 `""`                        | 返回右側         | 返回 `""`      |
| `0`         | 編譯錯誤 (因為 0 是 non-nullable)     | 返回右側         | 返回 `0`       |
| `false`     | 編譯錯誤 (因為 false 是 non-nullable) | 返回右側         | 返回 `false`   |
| `"hello"`   | 返回 `"hello"`                   | 返回 `"hello"` | 返回 `"hello"` |

### C# 範例

```csharp
string a = null ?? "default";      // "default"
string b = "" ?? "default";        // ""（空字串不是 null）
string c = "hello" ?? "default";   // "hello"

// 0 不能用 ??，因為 int 不可為 null
int? d = null ?? 100;              // 100
int? e = 50 ?? 100;                // 50
```

### JavaScript `||` 範例

```javascript
let a = null || "default";      // "default"
let b = undefined || "default"; // "default"
let c = "" || "default";        // "default" ⚠️ 空字串被視為 falsy
let d = 0 || 100;               // 100      ⚠️ 0 被視為 falsy
let e = false || true;          // true     ⚠️ false 被視為 falsy
let f = "hello" || "default";   // "hello"
```

### JavaScript `??` 範例（ES2020）

```javascript
let a = null ?? "default";      // "default"
let b = undefined ?? "default"; // "default"
let c = "" ?? "default";        // ""    ✅ 保留空字串
let d = 0 ?? 100;               // 0     ✅ 保留 0
let e = false ?? true;          // false ✅ 保留 false
let f = "hello" ?? "default";   // "hello"
```

---
## 賦值版本比較

|  運算符  | 語言  | 用途                             |
| :---: | :-: | ------------------------------ |
|  ??=  | C#  | 當變數為 `null` 時才賦值               |
|  ??=  | JS  | 當變數為 `null` 或 `undefined` 時才賦值 |
| \|\|= | JS  | 當變數為 falsy 時才賦值                |
|       |     |                                |

### C# `??=` 範例

```csharp
string? name = null;
name ??= "Guest";    // name 現在是 "Guest"

string? title = "Engineer";
title ??= "Unknown"; // title 仍然是 "Engineer"
```

### JavaScript `||=` 與 `??=` 範例

```javascript
// ||= 賦值
let a = null;
a ||= "default";     // a = "default"

let b = "";
b ||= "default";     // b = "default" ⚠️ 空字串被覆蓋

// ??= 賦值
let c = null;
c ??= "default";     // c = "default"

let d = "";
d ??= "default";     // d = "" ✅ 空字串被保留
```

---
## JavaScript 檢查方式總整理

| 寫法              | 檢查對象                 | 備註                    |
| --------------- | -------------------- | --------------------- |
| x != null       | `null` + `undefined` | 寬鬆相等                  |
| x !== null      | 僅 `null`             | 嚴格相等                  |
| x !== undefined | 僅 `undefined`        | 嚴格相等                  |
| x \|\| default  | 所有 falsy             | 包含 `0`, `""`, `false` |
| x ?? default    | `null` + `undefined` | ES2020                |

### `!= null` 範例

```javascript
// != null 同時檢查 null 和 undefined
let a = null;
let b = undefined;
let c = "";
let d = 0;

console.log(a != null);  // false（a 是 null）
console.log(b != null);  // false（b 是 undefined）
console.log(c != null);  // true （空字串不是 null/undefined）
console.log(d != null);  // true （0 不是 null/undefined）

// 常見用法：確認變數有值才執行
if (userName != null) {
    console.log("Hello, " + userName);
}
```

### 為什麼 `!= null` 能同時檢查兩者？

```javascript
// JavaScript 的寬鬆相等中，null 和 undefined 互等
null == undefined   // true
null === undefined  // false（嚴格相等則不同）
```

---
## 實務建議

| 情境                          |  推薦用法   |
| --------------------------- | :-----: |
| 想保留 `0` 或空字串 `""`           |   ??    |
| 想把所有假值都替換掉                  |  \|\|   |
| 檢查變數是否有值（排除 null/undefined） | != null |
| 只在變數無值時賦予預設值                |   ??=   |

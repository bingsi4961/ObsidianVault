---
title: JavaScript 字典(Dictionary) 完整筆記
tags: [JavaScript]

---

# 1. JavaScript 字典的概念與實現

## 1.1 什麼是字典？
字典是一種鍵值對（key-value）的資料結構，允許通過唯一的鍵快速查找對應的值。在 JavaScript 中，有兩種主要方式實現字典：
- 使用物件（Objects）
- 使用 Map 對象

## 1.2 使用物件實現字典

### 1.2.1 創建字典
```javascript
// 使用物件字面量創建空字典
const emptyDict = {};

// 使用物件字面量創建帶有初始值的字典
const person = {
    name: "張三",
    age: 25,
    city: "台北"
};

// 使用 Object 構造函數創建字典
const dict = new Object();
dict.key1 = "值1";
dict["key2"] = "值2";
```

### 1.2.2 添加和修改鍵值對
```javascript
const dict = {};

// 使用點符號添加/修改
dict.name = "李四";

// 使用方括號添加/修改（更靈活，可使用變數或含特殊字元的鍵）
dict["age"] = 30;

// 使用變數作為鍵
const key = "email";
dict[key] = "li4@example.com";

// 含空格或特殊字元的鍵必須使用方括號
dict["home address"] = "台北市信義區";
```

### 1.2.3 訪問值
```javascript
// 使用點符號
console.log(dict.name);  // 輸出: "李四"

// 使用方括號（更靈活）
console.log(dict["age"]);  // 輸出: 30

// 使用變數作為鍵
const prop = "email";
console.log(dict[prop]);  // 輸出: "li4@example.com"

// 訪問不存在的屬性會返回 undefined
console.log(dict.nonExistent);  // 輸出: undefined
```

### 1.2.4 刪除鍵值對
```javascript
delete dict.age;  // 刪除 age 鍵及其值
```

## 1.3 使用 Map 實現字典

### 1.3.1 創建 Map 字典
```javascript
// 創建空 Map
const userMap = new Map();

// 創建帶有初始值的 Map
// Map 只能轉換二維陣列的資料，不能轉換物件
const initialMap = new Map([
    ["name", "王五"],
    ["age", 28]
]);
```

### 1.3.2 添加和修改鍵值對
```javascript
// 添加或修改鍵值對
userMap.set("name", "王五");
userMap.set("age", 28);
userMap.set(42, "數字可以作為鍵");  // Map 的鍵可以是任何類型

// 鏈式調用
userMap.set("city", "高雄").set("isActive", true);
```

### 1.3.3 訪問值
```javascript
console.log(userMap.get("name"));  // 輸出: "王五"
console.log(userMap.get("nonExistent"));  // 輸出: undefined
```

### 1.3.4 刪除鍵值對
```javascript
userMap.delete("age");  // 返回 true 如果鍵存在並被成功刪除

// 清空所有鍵值對
userMap.clear();
```

### 1.3.5 Map 的其他操作
```javascript
// 獲取 Map 的大小
console.log(userMap.size);

// 檢查鍵是否存在
console.log(userMap.has("name"));  // 輸出 true 或 false
```

## 1.4 物件 vs. Map 作為字典的比較

| 特性 | 物件 | Map |
|------|------|-----|
| 鍵的類型 | 僅字符串或符號 | 任何值（包括物件、函數等） |
| 大小獲取 | 需使用 Object.keys(objDict).length | 直接使用 map.size |
| 迭代順序 | 不保證順序（除非使用特殊技巧） | 與插入順序一致 |
| 默認鍵 | 有原型鍵（除非使用 Object.create(null)） | 無默認鍵 |
| 性能 | 一般來說適用於小型字典 | 適用於頻繁增刪的大型字典 |

## 1.5 遍歷字典

### 1.5.1 使用 for...in 循環

```javascript
for (let key in dict) {
    if (dict.hasOwnProperty(key)) {  // 確保只處理自身屬性
        console.log(`${key}: ${dict[key]}`);
    }
}
```

### 1.5.2 使用 Object.keys()、Object.values() 和 Object.entries()

```javascript
// 獲取所有鍵 (Property Name)
const keys = Object.keys(dict);
console.log(keys);  // 輸出: ["name", "email", "home address"]

// 獲取所有值 (Property Value)
const values = Object.values(dict);
console.log(values);  // 輸出: ["李四", "li4@example.com", "台北市信義區"]

// 獲取所有鍵值對 (轉換成二維陣列)
const entries = Object.entries(dict);
console.log(entries);  // 輸出: [["name", "李四"], ["email", "li4@example.com"], ["home address", "台北市信義區"]]

// 使用 forEach 遍歷
Object.entries(dict).forEach(([key, value]) => {
    console.log(`${key}: ${value}`);
});
```
### 1.5.3 遍歷 Map()

```javascript
// 遍歷 Map
userMap.forEach((value, key) => {
    console.log(`${key}: ${value}`);
});
```


# 2. 理解 JavaScript 中的真值和假值

## 2.1 JavaScript 的假值
在使用 `if(!dictionary[key])` 這類表達式時，理解 JavaScript 的真值和假值非常重要。

以下值在布爾上下文中被視為 `false`：
- `false`
- `0`（數字零）
- `-0`（負零）
- `0n`（BigInt 零）
- `""`（空字串）
- `null`
- `undefined`
- `NaN`（非數字）

## 2.2 JavaScript 的真值
除了上述假值外，其他所有值都被視為 `true`，包括：
- 所有物件（包括空物件 `{}`）
- 所有陣列（包括空陣列 `[]`）
- 所有非空字串
- 所有非零數字
- 日期對象
- 函數
- 正則表達式


# 3. 字典中的屬性檢查

## 3.1 檢查屬性是否存在的方法

### 3.1.1 使用 `in` 運算符
```javascript
if ("name" in dictionary) {
    console.log("鍵 'name' 存在");
}

if (!(key in dictionary)) {
    // 鍵不存在
    dictionary[key] = initialValue;
}
```
**特點**：
- 檢查屬性是否存在於物件或其原型鏈中
- 會檢查繼承的屬性
- ==不考慮屬性的值==

### 3.1.2 使用 `hasOwnProperty` 方法
```javascript
if (dictionary.hasOwnProperty("name")) {
    console.log("字典中有自己的 'name' 屬性");
}

if (!dictionary.hasOwnProperty(key)) {
    // 鍵不存在
    dictionary[key] = initialValue;
}
```
**特點**：
- 僅檢查物件自身是否具有該屬性（不考慮繼承的屬性）
- ==不考慮屬性的值==
- ==可能被覆蓋，如果物件自身定義了同名方法==

### 3.1.3 使用 `Object.prototype.hasOwnProperty.call`
```javascript
if (Object.prototype.hasOwnProperty.call(dictionary, "name")) {
    console.log("字典中有自己的 'name' 屬性");
}
```
**特點**：
- 比 `hasOwnProperty` 更安全，防止方法被覆蓋
- 可用於沒有 `hasOwnProperty` 方法的物件

### 3.1.4 使用 `Object.hasOwn`（ES2022 新增）
```javascript
if (Object.hasOwn(dictionary, "name")) {
    console.log("字典中有自己的 'name' 屬性");
}
```
**特點**：
- 更簡潔的語法
- 解決了 `hasOwnProperty` 的安全問題
- 能正確處理 `null` 原型物件
- 需要較新的 JavaScript 環境支持

### 3.1.5 直接使用屬性值檢查
```javascript
// 如果 name 屬性不存在，dictionary.name 會是 undeined
if (dictionary.name !== undefined) {
    console.log("鍵 'name' 存在且值不是 undefined");
}

// 更常見的形式
if (!dictionary[key]) {
    // 鍵不存在或值是假值
    dictionary[key] = initialValue;
}
```
**特點**：
- ==實際上檢查的是「屬性值是否不是 undefined」，不是純粹的存在性檢查==
- 如果屬性存在但值為 `undefined`，會給出假陽性結果
- 使用否定形式時，會將假值屬性（`0`、`""`、`false`、`null`）視為不存在

## 3.2 `!dictionary[key]` 行為解析

當我們使用 `if(!dictionary[key])` 這種條件檢查時，JavaScript 會根據不同情況產生不同的結果。讓我詳細解釋這兩種情況：

### 3.2.1 情況一：屬性完全不存在

當你嘗試訪問一個對象中不存在的屬性時，JavaScript 會返回 `undefined`：

```javascript
const dictionary = {};
console.log(dictionary.nonExistentKey);  // 輸出：undefined
console.log(dictionary["nonExistentKey"]);  // 輸出：undefined
```

套用到條件判斷中：

```javascript
if(!dictionary["nonExistentKey"]) {
  console.log("這段代碼會執行，因為 !undefined 等於 true");
}
```

這是因為 `undefined` 是 JavaScript 中的假值（falsy value `undefined`、`null`、`0`、`""`、`false`），而 `!undefined` 評估為 `true`。

### 3.2.2 情況二：屬性存在但值未指定

這種情況可能有幾種不同的變體：

1. 屬性明確設為 `undefined`

```javascript
const dictionary = { someKey: undefined };
console.log(dictionary.someKey);  // 輸出：undefined
console.log("someKey" in dictionary);  // 輸出：true（屬性確實存在）

if(!dictionary.someKey) {
  console.log("這段代碼會執行，因為 !undefined 等於 true");
}
```

2. 屬性設為 `null`

```javascript
const dictionary = { someKey: null };
console.log(dictionary.someKey);  // 輸出：null

if(!dictionary.someKey) {
  console.log("這段代碼會執行，因為 !null 等於 true");
}
```

3. 屬性設為其他假值

```javascript
const dictionary = { 
  emptyString: "", 
  zero: 0, 
  falseBoolean: false
};

// 這些條件都會評估為 true，導致代碼區塊被執行
if(!dictionary.emptyString) { /* 會執行 */ }
if(!dictionary.zero) { /* 會執行 */ }
if(!dictionary.falseBoolean) { /* 會執行 */ }
```

4. 屬性設為真值

```javascript
const dictionary = { 
  nonEmptyString: "hello", 
  nonZeroNumber: 42, 
  trueBoolean: true,
  object: {},
  array: []
};

// 這些條件都會評估為 false，代碼區塊不會執行
if(!dictionary.nonEmptyString) { /* 不會執行 */ }
if(!dictionary.nonZeroNumber) { /* 不會執行 */ }
if(!dictionary.trueBoolean) { /* 不會執行 */ }
if(!dictionary.object) { /* 不會執行 */ }
if(!dictionary.array) { /* 不會執行，因為空陣列是真值！ */ }
```

### 3.2.3 關鍵區別：檢查屬性存在性 vs. 檢查屬性值

使用 `if(!dictionary[key])` 實際上是在檢查 ==「該屬性的值是否為假值」，而不是檢查「該屬性是否存在」==。這是一個重要的區別：

```javascript
const dictionary = { 
  exists: undefined,  // 屬性存在但值為 undefined
  zero: 0             // 屬性存在但值為 0（假值）
};

// 檢查屬性值是否為假值
if(!dictionary.exists) { /* 會執行，因為 !undefined 為 true */ }
if(!dictionary.zero) { /* 會執行，因為 !0 為 true */ }
if(!dictionary.missing) { /* 會執行，因為 !undefined 為 true */ }

// 檢查屬性是否存在
if(!("exists" in dictionary)) { /* 不會執行，因為屬性確實存在 */ }
if(!dictionary.hasOwnProperty("zero")) { /* 不會執行，因為屬性確實存在 */ }
if(!(Object.hasOwn(dictionary, "missing"))) { /* 會執行，因為屬性不存在 */ }
```

### 3.2.4 ★在初始化字典時的最佳實踐

在您原始函數的應用場景中（初始化一個陣列作為字典值），使用 `if(!dictionary[key])` 是可行的，因為：

1. ==如果屬性不存在，`dictionary[key]` 會是 `undefined`，`!undefined` 為 `true`，條件成立==
2. ==如果屬性已存在且值是陣列（即使是空陣列），`![]` 為 `false`，條件不成立，不會重新初始化==

然而，如果您的代碼可能處理值為假值（如 `0`、`""`、`false`）的屬性，或者您想明確表達「檢查屬性是否存在」的意圖，那麼使用 `if(!dictionary.hasOwnProperty(key))` 或 `if(!(key in dictionary))` 會更加準確。

```javascript
// 更準確的版本
function organizeDictionaryBySeriesName(data) {
    const dictionary = {};
    
    $.each(data, function(index, item) {
        const key = item.seriesName;
        
        // 檢查屬性是否存在及屬性值是否為陣列        
        if(!dictionary.hasOwnProperty(key) || !Array.isArray(dictionary[key])) {
            dictionary[key] = [];
        }
        
        dictionary[key].push({
            groupId: item.groupId,
            groupName: item.groupName
        });
    });
    
    return dictionary;
}
```

這樣即使屬性值是假值，也能正確處理，並且代碼意圖更加明確。



## 3.3 各種檢查方法的行為比較

下表詳細比較不同屬性檢查方法在各種情況下的行為：

| 情況 | obj[key] | !obj[key] | key in obj | obj.hasOwnProperty(key) | Object.hasOwn(obj, key) |
|------|----------|-----------|------------|--------------------------|--------------------------|
| 屬性不存在 | undefined | true | false | false | false |
| 屬性存在，值為 undefined | undefined | true | true | true | true |
| 屬性存在，值為 null | null | true | true | true | true |
| 屬性存在，值為 0 | 0 | true | true | true | true |
| 屬性存在，值為 "" | "" | true | true | true | true |
| 屬性存在，值為 false | false | true | true | true | true |
| 屬性存在，值為 true | true | false | true | true | true |
| 屬性存在，值為非零數字 | 數字值 | false | true | true | true |
| 屬性存在，值為非空字串 | 字串值 | false | true | true | true |
| 屬性存在，值為物件（包括空物件） | 物件參考 | false | true | true | true |
| 屬性存在，值為陣列（包括空陣列） | 陣列參考 | false | true | true | true |
| 屬性存在於原型鏈中 | 值或 undefined | 視值而定 | true | false | false |

# 4. 實際應用範例

## 4.1 使用物件實現分組函數

```javascript
function organizeDictionaryBySeriesName(data) {
    // 創建空字典
    const dictionary = {};
    
    // 遍歷資料陣列
    data.forEach(function(item) {
        const key = item.seriesName;
        
        /*
        // 使用 hasOwnProperty 進行更精確的檢查
        if (!dictionary.hasOwnProperty(key)) {
        
        // 使用更安全的檢查方法，防止 hasOwnProperty 被覆蓋
        if (!Object.prototype.hasOwnProperty.call(dictionary, key)) {
                    
        // 使用現代的 Object.hasOwn 方法
        if (!Object.hasOwn(dictionary, key)) {
        
        // 若屬性不存在(undefined) 或屬性值為 falsy value (null、undefined、0、false、NaN ) 就執行
        if (!dictionary[key]) {
            dictionary[key] = [];
        }        
        // 在前面的判斷式，假設屬性存在且屬性值為 true value，就不會執行
        // 但當屬性值不是陣列型態，執行後面 push 程式時，會發生錯誤
        */
        
        if(!dictionary.hasOwnProperty(key) || !Array.isArray(dictionary[key])) {
            dictionary[key] = [];            
        }
        
        // 將當前項目的 groupId 和 groupName 添加到對應鍵的陣列中
        dictionary[key].push({
            groupId: item.groupId,
            groupName: item.groupName
        });
    });
    
    return dictionary;
}

// 使用範例
const data = [
    { seriesName: "A系列", groupId: 1, groupName: "A1組" },
    { seriesName: "A系列", groupId: 2, groupName: "A2組" },
    { seriesName: "B系列", groupId: 3, groupName: "B1組" }
];

const result = organizeDictionaryBySeriesName(data);
console.log(result);

/* 輸出:
{
    "A系列": [
        { groupId: 1, groupName: "A1組" },
        { groupId: 2, groupName: "A2組" }
    ],
    "B系列": [
        { groupId: 3, groupName: "B1組" }
    ]
}
*/
```

## 4.2 使用 Map 實現分組函數

```javascript
function organizeWithMap(data) {
    const resultMap = new Map();
    
    data.forEach((item) => {
        const key = item.seriesName;
        
        // 檢查 Map 中是否已有該鍵
        if (!resultMap.has(key) || !Array.isArray(resultMap.get(key))) {
            resultMap.set(key, []);
        }
        
        // 取得當前陣列並添加新項目
        resultMap.get(key).push({
            groupId: item.groupId,
            groupName: item.groupName
        });
    });
    
    return resultMap;
}

// 使用範例
const mapResult = organizeWithMap(data);
console.log(mapResult);

/* 輸出是一個 Map 物件：
Map(2) {
  "A系列" => [
    { groupId: 1, groupName: "A1組" },
    { groupId: 2, groupName: "A2組" }
  ],
  "B系列" => [
    { groupId: 3, groupName: "B1組" }
  ]
}
*/

// 如果需要轉換回普通物件
const objectFromMap = Object.fromEntries(mapResult);
console.log(objectFromMap);
```
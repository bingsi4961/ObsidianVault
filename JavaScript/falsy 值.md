---
title: falsy 值
tags: [JavaScript]

---

以下是所有的 falsy 值：

```javascript
false          // 布林值 false
0              // 數字零
-0             // 負零
0n             // BigInt 零
""             // 空字串
''             // 空字串（單引號）
``             // 空字串（反引號）
null           // null
undefined      // undefined
NaN            // 非數字
```

讓我們通過實際例子來理解：

```javascript
console.log(false || 'default')    		// 輸出: "default"
console.log(0 || 'default')        		// 輸出: "default"
console.log(-0 || 'default')       		// 輸出: "default"
console.log(0n || 'default')       		// 輸出: "default"
console.log("" || 'default')       		// 輸出: "default"
console.log('' || 'default')       		// 輸出: "default"
console.log(`` || 'default')       		// 輸出: "default"
console.log(null || 'default')     		// 輸出: "default"
console.log(undefined || 'default')           // 輸出: "default"
console.log(NaN || 'default')     		// 輸出: "default"

if (false)      { console.log('不會執行'); }
if (0)          { console.log('不會執行'); }
if (-0)         { console.log('不會執行'); }
if (0n)         { console.log('不會執行'); }
if ("")         { console.log('不會執行'); }
if ('')         { console.log('不會執行'); }
if (``)         { console.log('不會執行'); }
if (null)       { console.log('不會執行'); }
if (undefined)  { console.log('不會執行'); }
if (NaN)        { console.log('不會執行'); }
```



空值合併運算符（??）與 falsy 的差異：
```javascript
// || 運算符檢查 falsy
console.log(0 || 'default');       // 輸出: "default"
console.log('' || 'default');      // 輸出: "default"

// ?? 運算符只檢查 null 或 undefined
console.log(0 ?? 'default');       // 輸出: 0
console.log('' ?? 'default');      // 輸出: ""
console.log(null ?? 'default');    // 輸出: "default"
console.log(undefined ?? 'default');// 輸出: "default"
```

注意事項：

1. 所有其他值都是 truthy（會被轉換為 `true`），包括：
   - 任何非空字串
   - 任何非零數字
   - 物件 `{}`
   - 陣列 `[]`
   - 函式 `function(){}`

2. 在判斷特定值時，最好使用嚴格相等（===）而不是依賴 falsy 轉換：
```javascript
// 不太好的寫法
if (!value) { }

// 更好的寫法（如果你確實要檢查 null 或 undefined）
if (value === null || value === undefined) { }
```

3. 使用 falsy 檢查時要小心數字 0 和空字串的情況：
```javascript
function process(value) {
    // 如果 0 是有效值，這樣的檢查可能會有問題
    if (!value) {
        return 'invalid';
    }
    return 'valid';
}

console.log(process(0));  // 輸出: "invalid"，但可能不是你想要的結果
```

---
空物件 `{}` 和空陣列 `[]` 都是 truthy 值。讓我們透過一些例子來驗證：

```javascript
// 空物件
if ({}) {
    console.log('空物件是 truthy');  // 會執行
}

// 空陣列
if ([]) {
    console.log('空陣列是 truthy');  // 會執行
}

// 實際應用中常見的錯誤
const arr = [];
if (!arr) {
    console.log('陣列是空的');  // 不會執行，因為 [] 是 truthy
}

// 正確檢查空陣列的方法
if (arr.length === 0) {
    console.log('陣列是空的');  // 會執行
}

// 檢查空物件
const obj = {};
if (!obj) {
    console.log('物件是空的');  // 不會執行，因為 {} 是 truthy
}

// 正確檢查空物件的方法
if (Object.keys(obj).length === 0) {
    console.log('物件是空的');  // 會執行
}
```

所以，要檢查陣列或物件是否為空，應該：

1. 檢查陣列是否為空：
```javascript
const arr = [];
// 方法 1：檢查長度
if (arr.length === 0) { }

// 方法 2：使用 Array.isArray() 結合長度檢查
if (Array.isArray(arr) && arr.length === 0) { }
```

2. 檢查物件是否為空：
```javascript
const obj = {};
// 方法 1：使用 Object.keys()
if (Object.keys(obj).length === 0) { }

// 方法 2：使用 Object.entries()
if (Object.entries(obj).length === 0) { }
```

==這是初學者常見的一個誤區，因為直覺上會認為「空」的東西應該是 falsy，但在 JavaScript 中並非如此。==
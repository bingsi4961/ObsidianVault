---
date: 2025-07-06 09:54
aliases: 
tags:
  - JavaScript
  - Promise
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## 鏈式中的最後一個 `then()` 回傳 Promise 給呼叫的 function

無論鏈式中的最後一個 `.then()` 有沒有 `return`，整個 Promise 鏈都會回傳一個 Promise 給呼叫的 function。

**情況分析：**
```javascript
// 情況 1：最後一個 then 有 return
function myFunction() {
    return Promise.resolve('開始')
        .then(result => {
            console.log('處理中...');
            return '完成';  // 明確回傳值
        });
    // 整個鏈式回傳 Promise.resolve('完成')
}

// 情況 2：最後一個 then 沒有 return
function myFunction() {
    return Promise.resolve('開始')
        .then(result => {
            console.log('處理中...');
            // 隱式回傳 undefined
        });
    // 整個鏈式回傳 Promise.resolve(undefined)
}
```

**重要概念：**

-  `.then()` 永遠回傳一個新的 Promise
-  鏈式中的最後一個 `.then()` 決定整個鏈式的最終回傳值
-  無論有沒有明確 `return`，都會回傳 Promise 給呼叫的 function

## `catch()` 也會回傳 Promise 給呼叫的 function

執行到 .catch() 時，無論有沒有 return，整個 Promise 鏈都會回傳一個 Promise 給呼叫的 function。

**情況分析：**
```javascript
// 情況 1：catch 有 return
function myFunction() {
    return Promise.resolve('開始')
        .then(result => {
            throw new Error('發生錯誤');
        })
        .catch(error => {
            console.log('捕獲錯誤:', error.message);
            return '錯誤已處理';  // 明確回傳值
        });
    
    // 整個鏈式回傳 Promise.resolve('錯誤已處理')
    // ★★ 注意是 resolve 喔!!
}

// 情況 2：catch 沒有 return
function myFunction() {
    return Promise.resolve('開始')
        .then(result => {
            throw new Error('發生錯誤');
        })
        .catch(error => {
            console.log('捕獲錯誤:', error.message);
            // 隱式回傳 undefined
        });
    
    // 整個鏈式回傳 Promise.resolve(undefined)
    // ★★ 注意是 resolve 喔!!
}

// 情況 3：catch 中拋出新錯誤 
function myFunction() { 
	return Promise.resolve('開始')
		.then(result => { 
			throw new Error('第一個錯誤'); 
		})
		.catch(error => {
			console.log('捕獲第一個錯誤:', error.message); 
			throw new Error('第二個錯誤'); // 在 catch 中拋出新錯誤 
		}); 
		
	// 整個鏈式回傳 Promise.reject(new Error('第二個錯誤')) 
}
```

**重要概念：**

-  `.catch()` 也會回傳一個新的 Promise
-  ★★ 如果 `.catch()` 正常執行（不拋出錯誤），會回傳 ==**resolved Promise**==
-  ★★ 如果 `.catch()` 中又拋出錯誤，會回傳 ==**rejected Promise**==

## `finally()` 確實也會回傳一個 Promise，但它通常不會改變 Promise 的狀態和值。

**重要特性：**

- `.finally()` 會**透傳** ==前一個 Promise 的狀態和值==
- 不管 `.finally()` 中有沒有 `return`，==通常都不會影響最終結果==
- 除非在 `.finally()` 中拋出錯誤

**範例分析：**

```javascript
// 情況 1：finally 有 return（通常被忽略）
function myFunction() {
    return Promise.resolve('成功結果')
        .then(result => {
            console.log('處理:', result);
            return result;
        })
        .finally(() => {
            console.log('清理工作');
            return '這個回傳值會被忽略';  // 這個 return 不會影響結果
        });
    
    // 整個鏈式仍然回傳 Promise.resolve('成功結果')
}

// 情況 2：finally 沒有 return
function myFunction() {
    return Promise.resolve('成功結果')
        .then(result => {
            return result;
        })
        .finally(() => {
            console.log('清理工作');
            // 沒有 return，但不影響結果
        });
    
    // 整個鏈式仍然回傳 Promise.resolve('成功結果')
}
```

**錯誤情況也一樣透傳：**

```javascript
function myFunction() {
    return Promise.resolve('開始')
        .then(result => {
            throw new Error('發生錯誤');
        })
        .finally(() => {
            console.log('無論成功失敗都會執行');
            return '這個也會被忽略';
        });
        
    // 整個鏈式回傳 Promise.reject(new Error('發生錯誤'))
}
```

**唯一例外：在 finally 中拋出錯誤**

```javascript
function myFunction() {
    return Promise.resolve('成功結果')
        .finally(() => {
            throw new Error('finally 中的錯誤');
        });
    
    // 整個鏈式回傳 Promise.reject(new Error('finally 中的錯誤'))
}
```

##### **所以 `.finally()` 確實會回傳 Promise，但它的主要用途是執行清理工作，而不是改變 Promise 的結果！**

- 每個 `.then()`、`.catch()`、`.finally()` ==都會返回一個新的 Promise==
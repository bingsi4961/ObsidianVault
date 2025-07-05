---
date: 2025-07-05 10:10
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

## 核心概念

### Promise 鏈中的 return 機制

在 Promise 的 `.then()` 回呼函數中，當您 `return` 一個值時，這個值會**自動被包裝成一個已解決（resolved）的 Promise**，然後傳遞給下一個 `.then()`。

### 範例程式碼

```javascript
myPromise
    .then((result) => {
        console.log('成功：', result);
        return '處理後的資料-XXX';  // 這裡返回的字串會被自動包裝成 Promise
    })
    .then((newResult) => {
        console.log('進一步處理：', newResult);  // newResult 就是 '處理後的資料-XXX'
    })
    .catch((error) => {
        console.log('錯誤：', error);
    });
```

## 自動包裝機制

### 等價寫法

```javascript
// 原始寫法
return '處理後的資料-XXX';

// 等同於
return Promise.resolve('處理後的資料-XXX');
```

## 不同情況的處理

### 1. 返回一般值

```javascript
.then((result) => {
    return '普通字串';  // 被包裝成 resolved Promise
    // 等於 return Promise.resolve('普通字串');
})
```

### 2. 返回 Promise

```javascript
.then((result) => {
    return Promise.resolve('資料');  // 直接使用該 Promise
})
```

### 3. 拋出例外

```javascript
.then((result) => {
    throw new Error('發生錯誤');  // 被包裝成 rejected Promise，跳到 catch
    // 等於 return Promise.reject(new Error('發生錯誤'));
})
```

### 4. 沒有 return（重要！）

```javascript
.then((result) => {
    console.log('處理中...');
    // 沒有 return 語句
    // 等於 return Promise.resolve(undefined);
})
```

**等同於：**

```javascript
.then((result) => {
    console.log('處理中...');
    return undefined;  // 隱含的 return undefined
    // 等於 return Promise.resolve(undefined);
})
```

**進一步等同於：**

```javascript
.then((result) => {
    console.log('處理中...');
    return Promise.resolve(undefined);  // 自動包裝成成功的 Promise
})
```

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
```

**重要概念：**

-  `.catch()` 也會回傳一個新的 Promise
-  如果 `.catch()` 正常執行（不拋出錯誤），會回傳 ==**resolved Promise**==
-  如果 `.catch()` 中又拋出錯誤，會回傳 ==**rejected Promise**==

## 重要觀念

### undefined 不會觸發 catch

當 `.then()` 中沒有 `return` 時：

- 下一個 `.then()` 會收到 `undefined`
- 這仍然是一個**成功的 Promise**（resolved state）
- 會繼續執行下一個 `.then()`
- **不會**跳到 `.catch()`

### 範例驗證

```javascript
Promise.resolve('初始資料')
    .then((result) => {
        console.log('第一個 then：', result); // 輸出：第一個 then： 初始資料
        // 沒有 return，等同於 return undefined;
    })
    .then((result) => {
        console.log('第二個 then：', result); // 輸出：第二個 then： undefined
        console.log('仍然在 then 中執行！');
    })
    .catch((error) => {
        console.log('這裡不會執行'); // 不會執行
    });
```

## 會跳到 catch 的情況

### 1. 拋出例外

```javascript
.then((result) => {
    throw new Error('發生錯誤');  // 會跳到 catch
})
```

### 2. 返回 rejected Promise

```javascript
.then((result) => {
    return Promise.reject('拒絕');  // 會跳到 catch
})
```

### 3. 前面的 Promise 本身就是 rejected

```javascript
Promise.reject('錯誤')
    .then((result) => {
        // 這裡不會執行
    })
    .catch((error) => {
        // 會執行這裡
    });
```

## 總結

- 每個 `.then()` ==都會返回一個新的 Promise==
- 如果沒有明確 `return`，就是 `return undefined`
- `undefined` 會被自動包裝成 `Promise.resolve(undefined)`
- 這是一個**成功狀態**的 Promise，只是承載的值是 `undefined`
- 所以下一個 `.then()` 會正常執行，參數就是 `undefined`
- **重點：會自動回傳一個成功的 Promise，只是沒有值可以給下一個 Then**

這就是為什麼即使沒有 `return`，Promise 鏈仍然會繼續往下執行，而不會中斷或跳到 `.catch()` 的原因。
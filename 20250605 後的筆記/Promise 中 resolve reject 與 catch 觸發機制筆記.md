---
date: 2025-07-08 15:26
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
# 連結筆記
#### 📑 [[歡迎]]
#### 📑 [[歡迎]]

---

## 核心概念

**觸發 `catch` 的決定因素是 Promise 的狀態（resolved/rejected），而非回傳資料的型別**

## 重要原則

### ✅ 會觸發 catch 的情況

- 使用 `reject()` → Promise 進入失敗狀態
- 無論 `reject()` 回傳什麼資料型別都會觸發 catch

### ❌ 不會觸發 catch 的情況

- 使用 `resolve()` → Promise 進入成功狀態
- 即使 `resolve()` 回傳 Error 物件也不會觸發 catch

## 實際範例

### 情況 1：resolve(error) - 不觸發 catch

```javascript
function test1() {
    return new Promise((resolve, reject) => {
        const error = new Error("這是錯誤訊息");
        resolve(error);  // 成功狀態，不會觸發 catch
    });
}

async function example1() {
    try {
        const result = await test1();
        console.log("取得結果:", result);  // 會執行，result 是 Error 物件
    } catch (error) {
        console.log("不會執行到這裡");
    }
}
```

### 情況 2：reject(normalData) - 觸發 catch

```javascript
function test2() {
    return new Promise((resolve, reject) => {
        const normalData = { name: "John", age: 30 };
        reject(normalData);  // 失敗狀態，會觸發 catch
    });
}

async function example2() {
    try {
        const result = await test2();
        console.log("不會執行到這裡");
    } catch (error) {
        console.log("進入 catch:", error);  // 會執行，error 是正常物件
    }
}
```

## 在 Ajax 封裝中的應用

```javascript
function ajaxPromise(options) {
    return new Promise((resolve, reject) => {
        $.ajax(options)
            .done((data, textStatus, jqXHR) => {
                resolve(data);  // 成功時回傳資料
            })
            .fail((jqXHR, textStatus, errorThrown) => {
                const error = new Error(`Ajax Error: ${jqXHR.status} ${jqXHR.statusText}`);
                error.jqXHR = jqXHR;
                reject(error);  // 失敗時拋出錯誤
            });
    });
}
```

## 關鍵要點

1. **Promise 狀態決定流程**
    
    - `resolve()` → `.then()` 或 `await` 正常執行
    - `reject()` → `.catch()` 或 `try-catch` 的 `catch` 區塊

2. **資料型別不影響流程**
    
    - `resolve(errorObject)` 仍然是成功狀態
    - `reject(normalData)` 仍然是失敗狀態

3. **`await` 的行為**
    
    - `await` 只是語法糖
    - 實際上依據 Promise 狀態決定是否拋出例外

## 實務建議

- 成功時使用 `resolve()` 回傳正常資料
- 失敗時使用 `reject()` 回傳 Error 物件
- 不要混用，避免邏輯混亂
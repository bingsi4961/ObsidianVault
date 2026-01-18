---
date: 2026-01-18 19:32
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
#### 📑 [[JavaScript Promise 完整教學（三）：進階技巧與並行處理]]

---

## 第一章：什麼是 Async / Await？

`async / await` 本質上就是 Promise 的**語法糖 (Syntactic Sugar)**，它讓你的程式碼從「回呼地獄」或「長長的鏈條」，變成優雅的小說。

### 1.1 核心概念

我們可以把它們看作一組搭檔：

- **`async`**：寫在函式前面，告訴程式：「這是一個包含非同步任務的函式」。
- **`await`**：寫在 Promise 前面，告訴程式：「請**停在這裡等**結果出來，拿到結果後再往下一行走」。

### 1.2 兩種寫法的對照表

|特性|Promise (.then)|async / await|
|---|---|---|
|**閱讀感**|鏈接式，像是一連串的指令|順序式，像是在讀一般的小說|
|**等待感**|透過回呼函式處理結果|程式碼看起來真的在「等待」|
|**錯誤處理**|使用 `.catch()`|使用傳統的 `try...catch`|

---

## 第二章：Async / Await 的執行流程

### 2.1 基本範例

```javascript
async function orderCoffee() {
    console.log("2. 開始點餐 (進入 async 函式)");

    // 遇到 await，程式會在這裡「暫停」
    // fetch 發出請求後，orderCoffee 函式會交出控制權，讓外面的程式繼續跑
    const response = await fetch("https://api.sampleapis.com/coffee/hot");
    const data = await response.json();

    console.log("4. 收到咖啡資料了！");
    return data;
}

console.log("1. 準備點餐");

orderCoffee(); // 注意：這裡不會等裡面跑完

console.log("3. 我去旁邊滑手機 (證明沒被卡住)");
```

**執行結果順序：`1 -> 2 -> 3 -> 4`**

### 2.2 為什麼順序是 1 → 2 → 3 → 4？

這就是 `await` 最神奇的地方，我們可以把它拆解成這幾個步驟：

1. **同步執行**：印出 `1`。
2. **進入非同步函式**：執行 `orderCoffee()`，印出 `2`。
3. **遇到 `await` (關鍵點)**：
    - `fetch` 被啟動了（發送網路請求）。
    - **魔法發生：** JavaScript 引擎會把 `orderCoffee` 函式裡面剩餘的程式碼（也就是印出 `4` 的部分）包裝成一個微任務，並放到一邊掛起來。
    - 然後，它會直接**跳出** `orderCoffee` 函式，回到主流程。
4. **繼續主流程**：印出 `3`。
5. **回頭處理**：當網路請求（fetch）完成，且 Call Stack 空了（同步程式碼跑完了），剛才被掛起來的「微任務」會被抓回來執行，最後印出 `4`。

### 2.3 `await` 會不會卡住整個瀏覽器？

**不會。** `await` 只會「暫停」那個 `async` 函式**內部**的執行，不會卡住整個瀏覽器的執行緒。後面的 `console.log("3. 我去滑手機")` 照樣會執行。

---

## 第三章：Resolve 在哪裡？

這是一個非常深入的問題：在 `async / await` 的世界裡，我們沒有看到 `resolve`，那它跑去哪裡了？

### 3.1 答案：被隱藏在「編譯器」與「JavaScript 引擎」的自動化流程中

在 `async / await` 的世界裡，`resolve` 分散在兩個地方：

1. **等待別人 resolve**：`await` 會在那裡盯著 `fetch` 回傳的 Promise，直到瀏覽器底層呼叫了那個 Promise 的 `resolve`。
2. **自己 resolve 給別人**：當你的 `async` 函式執行到 `return` 那一行時，它會自動呼叫它自己對外代表那個 Promise 的 `resolve`，通知「呼叫它的人」我跑完了。

### 3.2 編譯器如何轉換 async/await？

**你寫的代碼：**

```javascript
async function getCoffee() {
    const response = await fetch(url);
    console.log("4. 收到咖啡");
    return "完成";
}
```

**JavaScript 引擎眼中的代碼（模擬還原）：**

```javascript
function getCoffee() {
    // async 函式一定會回傳一個 Promise
    return new Promise((resolve, reject) => {
        
        // 遇到 await，其實是把後面的程式碼塞進 .then()
        const promiseFromFetch = fetch(url);

        promiseFromFetch.then((response) => {
            // --- 這裡就是「微任務」開始的地方 ---
            console.log("4. 收到咖啡");
            
            // 你的 return "完成"，實際上就是呼叫這個 async 函式的 resolve
            resolve("完成"); 
            // ---------------------------------------
        }).catch(err => reject(err));
        
    });
}
```

### 3.3 重點整理

|觀察點|說明|
|---|---|
|**第一個 Resolve**|來自 `fetch` 內部（瀏覽器底層）。當網路資料收齊時，瀏覽器會觸發該 Promise 的 `internal resolve()`。|
|**第二個 Resolve**|當 `async` 函式執行到 `return` 時，會自動呼叫對外那個 Promise 的 `resolve`。|

> **你可以把 `await` 想像成一個自動化的 `.then()` 產生器。** 它把你的程式碼從 `await` 那一行切開，前面是同步執行的，後面全部打包丟進 `.then()` 的回呼函式裡。

---

## 第四章：async 函式一定回傳 Promise

只要函式前面加上了 `async`，它就**一定會回傳一個 Promise**。即使你內容寫 `return "完成"`，JavaScript 也會把它包裝成 `Promise.resolve("完成")`。

```javascript
async function sayHello() {
    return "Hello";
}

// 等同於
function sayHello() {
    return Promise.resolve("Hello");
}

// 使用方式
sayHello().then(msg => console.log(msg)); // 印出 "Hello"
```

這保證了 `async` 函式的呼叫者永遠可以用 `.then()` 或 `await` 來處理它。

---

## 第五章：錯誤處理 - try...catch

使用 `async / await` 後，你可以直接用熟悉的 `try...catch` 語法來處理錯誤，而不必用 `.catch()`。

### 5.1 基本語法

```javascript
async function loginProcess() {
    try {
        const status = await checkAccount();
        console.log("帳號驗證成功：", status);
        
        const profile = await getProfilePic();
        console.log("圖片載入成功：", profile);
        
    } catch (error) {
        console.log("攔截到錯誤了：", error);
    }
}
```

### 5.2 `await` 處理錯誤的核心機制

當 `await checkAccount()` 執行且發生失敗時，過程並非「直接跳轉」，而是經過以下三個微觀步驟：

1. **產生拒絕 (Reject)**： `checkAccount()` 異步操作失敗，回傳一個狀態為 `rejected` 的 Promise 物件。
    
2. **運算子解析 (Unwrap)**： `await` 運算子負責「拆開」這個 Promise。當它發現狀態是 `rejected` 時，`await` 會在當前代碼行 **主動拋出（throw）** 一個異常。
    
3. **觸發攔截 (Catch)**： 因為 `await` 拋出了異常，控制權才會由 JavaScript 引擎轉交給最近的 `catch` 區塊。

4. **沒有 `await` 就沒有 `catch`**：如果不加 `await`，即便 Promise 失敗了，它也只是一個帶有錯誤訊息的物件，不會觸發 `try-catch` 的跳轉。

---

## 第六章：實戰改寫 - Async/Await 與 Promise 的互相轉換

### 6.1 從 Async/Await 轉換為 Promise 寫法

要把 `async/await` 的寫法轉換為傳統的 Promise (`.then`) 寫法，核心概念是將 `await` 後面的非同步操作串接到 `.then()` 回呼函式（Callback）中。

**Async/Await 版本：**

```javascript
async function orderCoffee() {
    console.log("2. 開始點餐");

    const resp = await fetch(url);
    const data = await resp.json();
    console.log("4. 收到咖啡資料了！");
    return data;
}
```

**轉換後的 Promise 版本：**

```javascript
function orderCoffee() {
    console.log("2. 開始點餐");

    // 返回 fetch 產生的 Promise 鏈
    return fetch(url)
        .then(function(resp) {
            // 第一層：取得 response 後，呼叫 .json()
            // 這也會回傳一個 Promise
            return resp.json();
        })
        .then(function(data) {
            // 第二層：取得解析後的 json 資料
            console.log("4. 收到咖啡資料了！");
            // 最後回傳資料，這會成為整個 Promise 鏈 resolve 的值
            return data;
        })
        .catch(function(error) {
            // 建議加上錯誤處理
            console.error("點餐發生錯誤：", error);
        });
}
```

**簡潔的箭頭函式版本 (ES6+)：**

```javascript
function orderCoffee() {
    console.log("2. 開始點餐");

    return fetch(url)
        .then(resp => resp.json())
        .then(data => {
            console.log("4. 收到咖啡資料了！");
            return data;
        });
}
```

### 6.2 關鍵差異對照

|項目|Async/Await 寫法|Promise 寫法|
|---|---|---|
|**函式宣告**|需要 `async` 關鍵字|普通函數即可回傳 Promise|
|**等待結果**|`const resp = await fetch(url)`|`fetch(url).then(resp => ...)`|
|**多步驟串接**|每個 `await` 各自一行|需要鏈式呼叫 `.then().then()`|
|**回傳值**|直接 `return data`，自動包裝成 Promise|需明確 `return fetch(...).then(...)`|
|**執行順序**|`console.log("2.")` 同步執行，`await` 後的程式碼等待結果|`console.log("2.")` 同步執行，後續邏輯必須寫在 `.then` 區塊內|

### 6.3 從 Promise 改寫為 Async/Await（登入流程）

**Promise 版本：**

```javascript
loginUser({ username: "user", password: "pass" })
    .then((loginResult) => {
        console.log("登入成功，Token:", loginResult.token);
        return fetchUserProfile(loginResult.token, loginResult.userId);
    })
    .then((userProfile) => {
        console.log("使用者資料載入完成：", userProfile);
    })
    .catch((error) => {
        console.error("操作過程發生錯誤：", error);
    });
```

**Async/Await 版本：**

```javascript
async function login() {
    try {
        const loginResult = await loginUser({ username: "user", password: "pass" });
        console.log("登入成功，Token:", loginResult.token);
        
        const userProfile = await fetchUserProfile(loginResult.token, loginResult.userId);
        console.log("使用者資料載入完成：", userProfile);
        
    } catch (error) {
        console.error("操作過程發生錯誤：", error);
    }
}

login();
```

你看，這段程式碼裡面沒有任何 `.then()`，順序就是 `登入 -> 取得資料`。這對於人類的大腦來說，理解負擔小很多！

### 6.4 搭配 Promise.all 使用

`async / await` 一樣可以搭配 `Promise.all` 來同時發送多個請求：

```javascript
async function loadDashboard() {
    try {
        // 同時發送三個請求
        const [userData, statistics, notifications] = await Promise.all([
            fetchUserData(123),
            fetchStatistics(),
            fetchNotifications()
        ]);
        
        console.log("全部載入完成！");
        updateUI(userData, statistics, notifications);
        
    } catch (error) {
        console.error("載入失敗：", error);
    }
}
```

---

## 第七章：與 C# 的對比

身為 C# 開發者，你會發現 JavaScript 的 `async / await` 與 C# 幾乎一模一樣：

|JavaScript|C#|說明|
|---|---|---|
|`async function`|`async Task` / `async Task<T>`|宣告非同步函式|
|`await fetch()`|`await HttpClient.GetAsync()`|等待非同步操作完成|
|`try...catch`|`try...catch`|錯誤處理方式相同|
|`Promise.all()`|`Task.WhenAll()`|等待所有任務完成|

### 7.1 狀態機 (State Machine)

JavaScript 的 `async / await` 編譯過程與 .NET 幾乎一模一樣。在 C# 中，編譯器會把 `async` 方法編譯成一個 `IAsyncStateMachine` 結構。

- `Task` 就像 `Promise`。
- `await` 就像 `TaskAwaiter`。
- **關鍵點**：當非同步作業完成時，會呼叫 `MoveNext()` 走進下一個狀態。在 JavaScript 中，這個「走進下一個狀態」的訊號，就是透過 Promise 的 `resolve` 觸發微任務佇列來達成的。

> **給 C# 工程師的小筆記：** JavaScript 的 `await` 其實就是把剩下的程式碼自動轉換成 `ContinueWith`。這也是為什麼它能保持 UI 不凍結，又能維持程式邏輯的先後順序。

---

## 第八章：常見問題與注意事項

### 8.1 `await` 只能在 `async` 函式內使用

```javascript
// ❌ 錯誤：await 不能在一般函式中使用
function getData() {
    const data = await fetch(url); // SyntaxError!
}

// ✅ 正確：必須在 async 函式中
async function getData() {
    const data = await fetch(url);
}
```

### 8.2 不要忘記 `await`

```javascript
async function example() {
    // ❌ 忘記寫 await，response 會是 Promise 物件而非結果
    const response = fetch(url);
    console.log(response); // Promise { <pending> }
    
    // ✅ 正確寫法
    const response = await fetch(url);
    console.log(response); // Response 物件
}
```

### 8.3 序列化執行 vs 並發啟動

```javascript
// 模式一：序列化執行 (Serial Execution)
async function sequential() {
    // 1. 執行 A 並暫停：主執行緒會等待 A 完成。
    // 2. 如果 A 是 CPU 運算，主執行緒會被 A 佔滿；如果是 I/O，主執行緒會閒置等待。
    const a = await taskA(); 
    
    // 只有在 A 徹底結束後，taskB 才會被呼叫。
    const b = await taskB(); 
    
    // 總耗時：永遠是 A + B。
}

// 模式二：並發啟動 (Concurrent Initiation)
async function parallel() {
    // 同時呼叫 taskA() 與 taskB()，嘗試將任務「同時」交給執行環境。
    const [a, b] = await Promise.all([taskA(), taskB()]);
    
    /**
     * 【效能差異關鍵筆記】
     * * 1. 如果是 I/O 密集 (如 Fetch API, Database)：
     * - 任務交由瀏覽器底層並行處理，主執行緒不參與等待。
     * - 總耗時 = max(A, B) -> 顯著變快。
     * * 2. 如果是 CPU 密集 (如 大量迴圈計算、加密、影像處理)：
     * - 雖然 Promise.all 嘗試並發，但 JS 主執行緒一次只能算一件事。
     * - A 算完之前，B 雖然被呼叫了，但只能在任務隊列排隊，無法動工。
     * - 總耗時 ≈ A + B -> 基本上與循序執行「沒有差別」。
     */
}
```

---

## 本章小結

在這一篇中，我們學習了：

1. `async / await` 的執行流程
2. Resolve 在哪裡？編譯器如何轉換
3. 錯誤處理：`try...catch`
4. 實戰改寫：從 Promise 到 Async/Await
5. 循序執行 vs 並行執行

---

[[JavaScript Promise 完整教學（三）：進階技巧與並行處理 | 上一篇：JavaScript Promise 完整教學（三）：進階技巧與並行處理]]
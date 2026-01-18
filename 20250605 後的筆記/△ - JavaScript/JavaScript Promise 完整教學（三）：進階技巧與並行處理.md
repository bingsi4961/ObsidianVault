---
date: 2026-01-18 15:21
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
#### 📑 [[JavaScript Promise 完整教學（二）：鏈式調用與回傳機制]]

---

## 第一章：Promise.resolve() 與 Promise.reject()

這兩個方法是 Promise 類別提供的**靜態方法（Static Methods）**。簡單來說，它們就是**「快速產生 Promise 物件的捷徑」**。

### 1.1 為什麼需要「捷徑」？

在之前的練習中，我們都是用 `new Promise((resolve, reject) => { ... })` 來建立 Promise。這通常用於處理「需要時間」的任務（例如 `setTimeout` 或 `$.ajax`）。

但有時候，你**手邊已經有資料了**，或者你**已經確定這件事失敗了**，你只是需要把它包裝成 Promise 的格式，好讓後續能接上 `.then()` 或 `.catch()`。這時用捷徑就會快很多。

**生活例子：預付卡與退貨單** 🎫

- **`Promise.resolve()`**：就像一張**儲值好的預付卡**。你不用等開卡流程，它裡面已經有錢了，拿到後立刻就能去消費（執行 `.then`）。
- **`Promise.reject()`**：就像一張**確定無效的退貨單**。你拿到它的一瞬間就知道它不能用，立刻就會去找客服處理（執行 `.catch`）。

### 1.2 語法對照表：手動 vs. 捷徑

|功能|傳統寫法 (長路)|捷徑寫法 (短路)|狀態|
|---|---|---|---|
|**立即成功**|`new Promise(res => res('OK'))`|**`Promise.resolve('OK')`**|**Fulfilled**|
|**立即失敗**|`new Promise((res, rej) => rej('Error'))`|**`Promise.reject('Error')`**|**Rejected**|

### 1.3 應用情境

**情境 A：快取（Cache）機制**

想像你在開發一個查詢天氣的系統。如果一分鐘內查過，就直接回傳舊資料，不用重新連網。

```javascript
function getWeatherData(city) {
    const cache = { "Taipei": "25°C" }; // 假設這是已經抓過的資料

    if (cache[city]) {
        // 資料已經有了，不需要 setTimeout，直接回傳一個成功的 Promise
        console.log("從快取中讀取...");
        return Promise.resolve(cache[city]);
    }

    // 如果沒有快取，才使用 new Promise 去連網抓資料
    return new Promise((resolve) => {
        console.log("正在連網抓取資料...");
        setTimeout(() => resolve("28°C"), 2000);
    });
}

// 不論是快取還是連網，用法都一模一樣
getWeatherData("Taipei").then(data => console.log("溫度是：" + data));
```

**情境 B：參數驗證**

如果你在函式開始時就發現參數不對，可以立刻回傳一個失敗的 Promise。

```javascript
function uploadFile(file) {
    if (!file) {
        // 立刻拋出失敗，不需要進入複雜的流程
        return Promise.reject("錯誤：未選擇檔案");
    }
    // ... 執行上傳邏輯
}

uploadFile(null).catch(err => console.log(err)); // 輸出：錯誤：未選擇檔案
```

### 1.4 `Promise.resolve()` 的隱藏超能力：轉換器

`Promise.resolve()` 有一個非常強大的特性：**它能把「不是 Promise 的東西」變成「Promise」。**

- 如果你傳入一個**一般數值**，它會回傳一個 Fulfilled 的 Promise。
- 如果你傳入一個**另一個 Promise**，它會直接回傳那個 Promise，不會重複包裝。
- 如果你傳入一個 **$.ajax (Thenable)**，它會把它轉換成標準的原生 Promise。

> **為什麼這很重要？** 當你不確定某個套件回傳的是不是標準 Promise 時，用 `Promise.resolve(結果)` 包一下，就能確保後續一定可以使用標準的 `.then()`。

---

## 第二章：同時處理多個 Promise

現實開發中，我們常常需要**同時處理多個非同步任務**。

想像你在準備一場聚會 🥳，你需要：

1. 訂比薩 🍕（20 分鐘）
2. 買飲料 🥤（10 分鐘）
3. 佈置場地 🎈（15 分鐘）

如果你「同步」做，要花 45 分鐘；如果你「非同步」同時並行，只需要 20 分鐘。這就是 `Promise.all` 等方法的威力。

### 2.1 Promise.all()：團結力量大（全成功才算數）

`Promise.all` 就像是一場**接力賽或團隊運動**：只要其中一個人跌倒（Reject），整隊就判定失敗。

**特點：**

- 並行執行所有任務。
- **成功時**：等最慢的那個完成後，回傳一個包含所有結果的**陣列**。
- **失敗時**：只要有一個人 `reject`，它會**立刻**跳到 `.catch()`，不管其他人跑完了沒。

**生活例子：出國旅行** ✈️

你跟三個朋友約好出國，所有人都拿到簽證（Resolve）才能出發。只要有一個人被拒簽（Reject），旅行就取消。

```javascript
const p1 = Promise.resolve("比薩到了");
const p2 = new Promise(res => setTimeout(() => res("飲料買好了"), 2000));
const p3 = Promise.resolve("場地佈置完畢");

Promise.all([p1, p2, p3])
    .then((results) => {
        // results 會是一個陣列：["比薩到了", "飲料買好了", "場地佈置完畢"]
        console.log("太棒了，聚會開始！", results);
    })
    .catch((err) => {
        console.log("慘了，有人出包：", err);
    });
```

**關於 `.catch()` 接收到的 `error`：**

當 `Promise.all` 失敗時，`error` 的確**只是那個失敗的 Promise 所傳出的訊息**，而不是陣列。`Promise.all` 追求的是「全有或全無」，只要有一個失敗，它就會立刻認為整個任務組都失敗了。它不會管其他還在跑的任務，直接把**第一個**遇到的錯誤丟給 `.catch()`。

### 2.2 Promise.allSettled()：大家辛苦了（不管成敗都要回報）

有時候你並不要求每件事都成功，你只是想知道「最後每件事的結果是什麼」。

**特點：**

- **永遠不會進入 `.catch()`**（除非程式本身壞掉）。它會等待所有任務「塵埃落定」。
- **回傳值**：一個物件陣列，告訴你每個任務的狀態（`fulfilled` 或 `rejected`）。

**生活例子：寄送電子郵件** 📧

你群發了 100 封信，有些成功、有些失敗（因為信箱不存在）。你不需要因為一封失敗就停止全部，你只需要最後給我一份報告。

```javascript
const req1 = Promise.resolve("郵件 A 成功");
const req2 = Promise.reject("郵件 B 失敗：位址錯誤");

Promise.allSettled([req1, req2])
    .then((results) => {
        results.forEach(res => {
            if (res.status === "fulfilled") {
                console.log("成功紀錄：", res.value);
            } else {
                console.log("失敗原因：", res.reason);
            }
        });
    });
```

**回傳結果的「長相」：**

```javascript
[
  { "status": "fulfilled", "value": "郵件 A 成功" },    // 成功：有 value
  { "status": "rejected",  "reason": "郵件 B 失敗：位址錯誤" }  // 失敗：有 reason
]
```

**結果物件的結構：**

|狀態 (status)|內容物 (Key)|什麼時候出現？|
|---|---|---|
|**`"fulfilled"`**|`value`: 成功的資料|任務順利完成時|
|**`"rejected"`**|`reason`: 失敗的原因|任務出錯時|

### 2.3 $.when()：jQuery 家族的並行器

這是 jQuery 版本的 `Promise.all`。雖然它很早就在用了，但它的運作邏輯跟原生 Promise 有點不太一樣（比較「老派」）。

**行為：** 類似 `Promise.all`，只要一個失敗就全失敗。

**關鍵差異：輸入與回傳格式**

- **輸入方式不同：** `Promise.all` 接收**陣列**；`$.when` 接收**分散的參數**。
- **回傳格式不同：** `Promise.all` 回傳**一個陣列**；`$.when` 回傳**多個獨立的參數**。

|比較項目|原生 `Promise.all`|jQuery `$.when`|
|---|---|---|
|**輸入方式**|`([p1, p2, p3])` (要括號，是陣列)|`(p1, p2, p3)` (不要括號，是分散的)|
|**處理多個回傳**|`(results) => { ... }` (拿一個陣列)|`(r1, r2, r3) => { ... }` (拿三個變數)|

**正確的 $.when 寫法：**

```javascript
const p1 = Promise.resolve('比薩到了');
const p2 = new Promise(res => setTimeout(() => res("飲料好了"), 2000));
const p3 = Promise.resolve('場地佈置完畢');

// 注意這裡沒有 [ ] 括號
$.when(p1, p2, p3)
    .done((res1, res2, res3) => {
        console.log("準備得如何了？", res1, res2, res3);
    })
    .fail(error => {
        console.log("慘了，有人出包：", error);
    });
```

**常見錯誤：傳入陣列**

```javascript
// ❌ 錯誤寫法：把陣列傳進去
$.when([p1, p2, p3])
    .done((res1, res2, res3) => {
        // res1 會變成整個陣列，res2 和 res3 會是 undefined
        console.log(res2[0]); // Uncaught TypeError: Cannot read properties of undefined
    });
```

當你寫 `$.when([p1, p2, p3])`（傳入陣列）時，jQuery 會把這整個陣列當成「第一個參數」。所以 `res2` 和 `res3` 都會是 `undefined`。

**如果一定要用陣列，可以使用展開運算子：**

```javascript
$.when(...[p1, p2, p3])
    .done((res1, res2, res3) => {
        console.log("聚會開始！", res1, res2, res3);
    });
```

### 2.4 回傳值的差異：`$.ajax` vs `new Promise`

使用 `$.when` 時，回傳值的「長相」取決於「是誰發出的」。

|發出任務的工具|回傳值的數量|`$.when().done()` 接收到的樣子|
|---|---|---|
|**`$.ajax()`**|**3 個** (data, status, xhr)|**陣列** `[data, status, xhr]`|
|**`new Promise()`**|**1 個** (你 resolve 的內容)|**單一值** (字串、物件或數字)|
|**`Promise.resolve()`**|**1 個**|**單一值**|

如果你的 `p1, p2, p3` 是用 `new Promise` 或 `Promise.resolve` 建立的（而非 `$.ajax`），那麼 `res1, res2, res3` 就不會是陣列 `[data, status, xhr]`，而是直接就是你 resolve 的那個值。

### 2.5 綜合比較表

|特性|`Promise.all()`|`Promise.allSettled()`|`$.when()`|
|---|---|---|---|
|**失敗處理**|**快速失敗**（有一壞就全壞）|**永不失敗**（等所有人做完）|**快速失敗**|
|**回傳格式**|乾淨的資料陣列|狀態與資料的物件陣列|散落的參數（args）|
|**執行順序**|同時發送（並行）|同時發送（並行）|同時發送（並行）|
|**適用時機**|任務之間有強烈依賴性|任務彼此獨立，只需最終報表|jQuery 專案且需相容舊程式|

> **建議：** 除非你的專案是舊有的、且大量依賴 jQuery，否則在寫新功能時，請一律使用 `Promise.all`。這能讓你的程式碼更具備未來性（Future-proof），且更符合現代開發的規範。

---

## 第三章：Promise.race()：競速執行

`Promise.race` 的特性是：**「只要有一棒先抵達終點（不論是成功還是失敗），就立刻結束並回傳該結果。」**

### 3.1 最常見的場景：逾時控制 (Timeout Control)

JavaScript 的 `fetch` 或某些版本的 AJAX，本身沒有內建方便的 Timeout 機制，我們可以自製一個：

```javascript
function fetchWithTimeout(url, timeout = 5000) {
    const fetchPromise = fetch(url);
    
    const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => {
            reject(new Error("請求逾時"));
        }, timeout);
    });
    
    // 看誰先完成：實際請求或逾時
    return Promise.race([fetchPromise, timeoutPromise]);
}

fetchWithTimeout('/api/slow-endpoint', 3000)
    .then(response => response.json())
    .then(data => console.log("資料載入成功：", data))
    .catch(error => console.error("載入失敗或逾時：", error));
```

### 3.2 背後原理

當你呼叫 `Promise.race([...])` 時：

1. **建立一個全新的「代理 Promise」**：這個 Promise 一開始的狀態是 `pending`（等待中）。
2. **訂閱所有參賽者**：這個代理 Promise 會同時去關注（訂閱）你傳入陣列中的所有 Promise。
3. **「誰先到，我就變成誰」**：只要陣列中任何一個 Promise 率先變成了 Settled 狀態（不論是 Resolved 還是 Rejected），代理 Promise 就會立刻「變身」：
    - 如果 `fetchPromise` 先成功：代理 Promise 會立刻變為 Resolved。
    - 如果 `timeoutPromise` 先失敗：代理 Promise 會立刻變為 Rejected。

### 3.3 進階觀念：輸掉的 Promise 會怎樣？

**「輸掉的人並不會停止執行」。** 瀏覽器依然會繼續下載資料直到完成或出錯，只是代理 Promise 已經不再理會它了。

---

## 第四章：使用 `Promise.resolve()` 轉換 `$.ajax`

### 4.1 核心規則：原生 Promise 的「單一值」原則

在 JavaScript 的標準規範中，一個 Promise 只能 `resolve` **一個值**。如果你傳給它三個參數，它也只會理會第一個，其他的都會被丟棄。

當你使用 `Promise.resolve($.ajax(...))` 時，JavaScript 引擎會發現 `$.ajax` 是一個 **Thenable**（具有 `.then` 方法的物件），它會自動去呼叫這個 `.then` 並等待結果。

### 4.2 結果會變成什麼？

當 `$.ajax` 成功時，它原本會傳回三個參數：`data`, `textStatus`, `jqXHR`。但透過 `Promise.resolve()` 轉換後，結果會發生以下變化：

|比較項目|jQuery `$.ajax().done()`|`Promise.resolve($.ajax())`|
|---|---|---|
|**回傳數量**|**三個** (`data`, `status`, `xhr`)|**一個** (`data`)|
|**結果內容**|伺服器回傳的資料、狀態字串、XHR 物件|**僅包含伺服器回傳的資料 (data)**|

### 4.3 如果需要全部資訊怎麼辦？

如果你需要 `textStatus` 或 `jqXHR` 來檢查標頭（Headers），請使用**手動封裝**：

```javascript
function fetchFullData() {
    return new Promise((resolve, reject) => {
        $.ajax({ url: '/api/test' })
            .done((data, status, xhr) => {
                // 手動包成一個物件，這樣下一棒就能拿到全部
                resolve({ data: data, status: status, xhr: xhr });
            })
            .fail(reject);
    });
}
```

---

## 第五章：Promise.all 的實戰應用

### 5.1 儀表板資料載入

```javascript
/**
 * 觀念：loadDashboardData 執行後會立即回傳一個 Promise 物件。
 * 類比：這就像 C# 中的 public async Task<DashboardData> LoadDashboardData()。
 */
const loadDashboardData = () => {
    // 1. 初始化三個非同步請求 (這時請求已經發出)
    const userDataPromise = fetchUserData(123);
    const statisticsPromise = fetchStatistics();
    const notificationsPromise = fetchNotifications();
    
    // 2. Promise.all 將多個 Promise 打包成一個
    // 它會回傳一個全新的 Promise，當全部成功時狀態為 Resolved，任一失敗則為 Rejected
    return Promise.all([userDataPromise, statisticsPromise, notificationsPromise])
        .then((results) => {
            // 觀念：results 的順序保證與傳入的陣列順序一致
            const [userData, statistics, notifications] = results;
            
            // 在這裡處理 UI 更新
            updateUserSection(userData);
            updateStatisticsSection(statistics);
            updateNotificationsSection(notifications);
            
            console.log("儀表板資料全部載入完成");

            /**
             * 觀念：必須使用 return 才能將資料傳遞給下一個 .then()
             * 如果不寫 return，下一個 .then 接收到的會是 undefined
             */
            return { userData, statistics, notifications }; 
        })
        .catch((error) => {
            /**
             * 觀念：這是「錯誤攔截」層
             * 如果這裡只寫 console.error 而沒有 throw，這個 Promise 鏈結會被視為「已修復」
             * 導致呼叫端的 .then() 被執行且接收到 undefined
             */
            console.error("載入儀表板資料時發生錯誤：", error);
            throw error; 
        });
};

/**
 * 呼叫端 (Caller) 的處理
 */
loadDashboardData()
    .then(data => { 
        console.log("成功拿到資料：", data.userData);
    })
    .catch(err => {
        showErrorModal("系統載入失敗，請稍後再試");
    });
```

---

## 第六章：單執行緒與併發（Concurrency）

### 6.1 JavaScript 的本質

**JavaScript 是「單執行緒 (Single-threaded)」，它實現的是「併發 (Concurrency)」，而不是「平行 (Parallelism)」。**

**核心概念釐清：**

- **單執行緒 (Single-threaded)**：JavaScript 引擎只有「一隻手」。它一次只能執行一行程式碼。
- **併發 (Concurrency)**：在同一段時間內，交替執行多個任務。看起來像是同時在跑，但實際上是在任務間快速切換。
- **平行 (Parallelism)**：在同一個瞬間，真的有多隻手同時在做不同的工作（通常需要多核心 CPU）。

### 6.2 Promise 是如何運作的？（以煮飯為例）

想像你正在廚房煮一頓大餐：

1. **單執行緒**：你是廚房裡唯一的廚師（只有你一個人）。
2. **併發 (Concurrency / Promise)**：
    - 你把肉放進烤箱（發出一個非同步請求）。
    - 你**不等**烤箱烤完，立刻轉身去切菜（執行下一行程式碼）。
    - 這時候，你其實並沒有同時在「烤肉」跟「切菜」，因為**烤箱（瀏覽器 API / 作業系統）**在幫你烤肉，而你的手在切菜。
    - 當烤箱響了（Promise Resolve），你切完菜後再去處理肉。
3. **平行 (Parallelism)**：這需要你請另一個廚師來。在 JavaScript 中，這叫 `Web Workers`，但普通的 Promise 並不是這樣。

### 6.3 Promise 的角色

Promise 只是負責**「管理這些外包任務的結果」**。

- **「執行」**任務的是瀏覽器（或系統）的底層線程（例如網路下載、計時器）。
- **「處理」**結果的是 JavaScript 的主線程。

|名稱|意義|JavaScript 中的例子|
|---|---|---|
|**併發 (Concurrency)**|雖然我只有一隻手，但我能管理很多同時進行的任務（不阻塞）。|**Promise**, `setTimeout`, AJAX|
|**平行 (Parallelism)**|我有多隻手，同一秒真的在運算兩個不同的東西。|**Web Workers** (JS 較少見的用法)|

### 6.4 兩個任務「同時」等待的原理

```javascript
.then(() => {
    任務 A; // 2秒的請求，主廚啟動烤箱後立刻抽手
    return 任務 B; // 3秒的請求，主廚啟動瓦斯爐後立刻抽手，並跟外面說「我在等瓦斯爐」
})
```

1. 主廚（執行緒）依序啟動 A 和 B，這過程不到 0.1 秒。
2. 接下來的 3 秒鐘，主廚是空閒的，可以去處理其他的網頁點擊事件。
3. 第 2 秒時，任務 A 完成了，但因為你沒 `return` 它，它的結果會被丟進垃圾桶。
4. 第 3 秒時，任務 B 完成了，主廚回來處理它，並交給下一個 `.then()`。

**結論**：非同步任務的「等待」是由瀏覽器後台處理的，所以 A 和 B 可以「同時等待」，不影響單執行緒的主廚。

---

## 第七章：與 C# 的對比

作為 C# 開發者，你會發現 JavaScript 的 `Promise` 與 `async/await` 的設計邏輯，與 C# 是**高度相似**的：

|JavaScript|C#|說明|
|---|---|---|
|`Promise`|`Task`|代表一個異步操作的未來結果|
|`await`|`await`|等待異步操作完成|
|`resolve / reject`|`TaskCompletionSource` 的結果設定|手動設定任務的結果|
|`Promise.all`|`Task.WhenAll`|等待所有任務完成|
|`Promise.race`|`Task.WhenAny`|等待任一任務完成|

最大的差別在於：JavaScript 是**單執行緒**運作的（透過 Event Loop），而 C# 的 `Task` 可能會在不同的執行緒（Thread Pool）上跑。但在寫法與流程控制上，你的 C# 經驗會讓你學 JavaScript 快非常多！

---

## 本系列總結

恭喜你完成了整個 JavaScript Promise 系列教學！你已經學會了：

**第一篇：基礎觀念與核心機制**

- 同步與非同步的差異
- Promise 的三種狀態（Pending、Fulfilled、Rejected）
- resolve/reject（動作）與 fulfilled/rejected（狀態）的區別
- 如何建立 Promise 以及 Promise 內容會立即執行的特性
- 如何使用 `.then()`、`.catch()`、`.finally()` 處理結果
- 實際應用：封裝 jQuery AJAX 為 Promise

**第二篇：鏈式調用與回傳機制**

- Promise 鏈式調用的運作原理
- `.then()` 的回傳機制（四種情境）
- Promise 展平 (Flattening)
- `.catch()` 的回傳機制（修復成功）
- `.finally()` 的回傳機制（透明過路站）
- 常見陷阱與實戰應用

**第三篇：進階技巧與並行處理**

- `Promise.resolve()` 與 `Promise.reject()` 快捷方法
- `Promise.all()`：全成功才算數
- `Promise.allSettled()`：不管成敗都要回報
- `$.when()`：jQuery 的並行器
- `Promise.race()`：競速執行與逾時控制
- JavaScript 的單執行緒與併發機制

你現在擁有的非同步知識，已經足以應付 95% 的實務開發需求！

---

## 下一步：Async / Await

既然你已經懂了：

1. Promise 是如何「管理任務」的。
2. 它們是如何在單執行緒中「併發」執行的。

現在，你可以進一步學習如何把這些複雜的 `.then()` 改寫成 **`async / await`**。

`async / await` 是 JavaScript 在 2017 年推出的終極語法，它能讓非同步程式碼「讀起來跟同步程式碼一模一樣」，本質上就是 Promise 的語法糖，讓你的程式碼從「回呼地獄」或「長長的鏈條」，變成優雅的小說。

---

[[JavaScript Promise 完整教學（二）：鏈式調用與回傳機制| 上一篇：JavaScript Promise 完整教學（二）：鏈式調用與回傳機制]]
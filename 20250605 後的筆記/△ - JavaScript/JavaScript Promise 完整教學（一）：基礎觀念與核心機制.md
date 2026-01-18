---
date: 2026-01-18 15:18
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

## 第一章：什麼是非同步？

在深入 Promise 之前，我們必須先理解「同步」與「非同步」的差別。

### 1.1 同步 vs 非同步

在程式的世界裡，大部分的情況是「一行執行完才執行下一行」，這叫做**同步 (Synchronous)**。但有些事情很花時間（例如：從網路下載一個大檔案），如果用同步的方式，整個網頁就會像當機一樣，動彈不得。

這時候我們需要**非同步 (Asynchronous)**，讓程式不會在等待時整個「卡死」在那裡。

|特性|同步 (Synchronous)|非同步 (Asynchronous)|
|---|---|---|
|**執行方式**|排隊制。前一個沒做完，後一個不能開始。|預約制。先交代任務，你可以先去做別的事。|
|**生活例子**|去銀行臨櫃排隊，沒輪到你之前只能在那裡等。|在美食街點餐拿呼叫器，你可以先去逛逛，等響了再去拿。|
|**優點**|直覺、好理解，順序固定。|不會卡住整個程式，效率高。|
|**缺點**|遇到耗時任務會讓程式「卡死」。|程式順序變得不直觀，需要特別處理結果。|

### 1.2 為什麼我們需要 Promise？

在沒有 Promise 之前，開發者必須使用一種叫「回呼函式 (Callback)」的方法。但當事情變得複雜（例如：先登入、再抓資料、最後存檔），程式碼會變得像一團亂掉的麵線（我們稱為「回呼地獄 Callback Hell」）。

Promise 的出現，就是為了讓程式碼讀起來像在說故事一樣：**「先做 A，成功後接著做 B，如果失敗就做 C」**。

---

## 第二章：什麼是 Promise？

Promise 就像是一個「承諾」或「約定」，它代表一個異步操作的最終完成或失敗。

### 2.1 生活化的比喻：咖啡廳點餐

想像一下，如果你在一家繁忙的咖啡廳點餐 ☕：

1. 你點完餐並付了錢，店員給了你一個**取餐呼叫器**。
2. 這張取餐呼叫器就是一個 **Promise**（承諾）：它承諾你，當餐點準備好時，它會通知你。
3. 在餐點還沒好之前，你不需要站在櫃台死守，你可以先去找座位、滑手機。
4. 當呼叫器震動時，代表「承諾」兌現了，你就可以去領咖啡。

在程式的世界裡，像「從網路下載圖片」或「從資料庫讀取資料」這種需要時間的事情，我們都會用 Promise 來處理，這樣程式就不會在等待時整個「卡死」在那裡。

---

## 第三章：Promise 的三種狀態

一個 Promise 在執行的過程中，一定會處於以下三種狀態之一：

|狀態|英文名稱|意義|生活例子|
|---|---|---|---|
|**等待中**|`Pending`|事情還在進行中，還沒有結果。|咖啡還在煮，呼叫器還沒響。|
|**已成功**|`Fulfilled`|事情順利完成。|咖啡煮好了，呼叫器震動。|
|**已失敗**|`Rejected`|發生錯誤，事情無法完成。|咖啡豆沒了，店員告訴你無法供應。|

### 3.1 狀態轉換的重要規則

Promise 的狀態是「不可逆」的單向道：

|初始狀態|執行的動作|最終狀態|後續影響|
|---|---|---|---|
|**Pending**|呼叫 `resolve()`|**Fulfilled** (成功)|觸發 `.then()`|
|**Pending**|呼叫 `reject()`|**Rejected** (失敗)|觸發 `.catch()`|
|**Pending**|**什麼都不做**|**Pending** (等待中)|**卡死，後續不執行**|

> **重要觀念**：Promise 一旦從 `Pending` 狀態轉換到 `Fulfilled` 或 `Rejected`，就無法再改變狀態。

### 3.2 釐清容易混淆的術語：動作 vs 狀態

很多開發者會把 `resolve/reject` 和 `fulfilled/rejected` 混在一起。簡單來說：

- **「Resolve / Reject」是動作（動詞）**：你主動呼叫的「函式」，用來發出通知。
- **「Fulfilled / Rejected」是狀態（形容詞）**：Promise 物件目前的「處境」，用來記錄結果。

|類別|成功系列|失敗系列|誰在使用？|意義|
|---|---|---|---|---|
|**動作 (Verbs)**|`resolve()`|`reject()`|**開發者（你）**|你主動呼叫的「函式」，用來發出通知。|
|**狀態 (States)**|`fulfilled`|`rejected`|**JavaScript 引擎**|Promise 物件目前的「處境」，用來記錄結果。|

**生活例子：快遞送貨** 🚚

1. **Pending（等待中）**：你下單了，手機還在路上。
2. **你的動作（決定結果的人）**：
    - `resolve`：快遞員把手機交到你手上。
    - `reject`：快遞員發現貨物掉了，打電話跟你說抱歉。
3. **訂單的最終狀態（結果）**：
    - `fulfilled`（已實現）：因為快遞員執行了 `resolve`，你的訂單狀態變成了「已完成」。
    - `rejected`（已拒絕）：因為快遞員執行了 `reject`，你的訂單狀態變成了「已取消」。

---

## 第四章：如何創造一個 Promise？

在 JavaScript 中，Promise 就像是一個「容器」，裡面裝著一個**未來才會完成**的任務。

### 4.1 基礎語法結構

我們用 `new Promise()` 來建立它。它需要一個函式作為參數，這個函式會**立即執行**，並接收兩個「通知官」：

1. **`resolve` (成功通知官)**：當事情順利完成時，呼叫他。
2. **`reject` (失敗通知官)**：當發生錯誤時，呼叫他。

```javascript
const coffeePromise = new Promise((resolve, reject) => {
    // 這裡模擬煮咖啡的時間...
    let isSuccess = true; // 假設煮咖啡成功了

    if (isSuccess) {
        resolve("☕ 咖啡煮好了！"); // 成功時，傳遞成功的訊息
    } else {
        reject("❌ 咖啡機壞了..."); // 失敗時，傳遞錯誤原因
    }
});
```

### 4.2 Promise 的執行時機

很多人會以為既然 Promise 是處理「非同步」的事情，那它裡面的程式碼應該會「等一下」才執行。但事實上：

> **Promise 裡面的函式（執行器 Executor）會「立即」執行。**

當你寫下 `new Promise(...)` 的那一瞬間，裡面的程式碼就已經開始跑了。只有「結果的回報」（也就是 `.then()` 或 `.catch()`）才會被放到工作排程中，等到之後才執行。

### 4.3 實驗：觀察執行的順序

```javascript
console.log("1. 準備點餐");

const coffeeOrder = new Promise((resolve, reject) => {
    console.log("2. 店員開始磨豆子（Promise 內部執行）");
    resolve("☕ 拿鐵");
});

console.log("3. 我去旁邊滑手機");

coffeeOrder.then((drink) => {
    console.log("4. 收到通知，拿到：" + drink);
});

console.log("5. 這裡是最後一行程式碼");
```

**執行結果的順序是：`1 -> 2 -> 3 -> 5 -> 4`**

|順序|步驟說明|原因|
|---|---|---|
|**1**|準備點餐|一般同步執行。|
|**2**|開始磨豆子|**Promise 的內容會立刻執行**，不會等待。|
|**3 & 5**|滑手機 & 最後一行|程式繼續往下跑，不會被 Promise 卡住。|
|**4**|收到通知|這是 `.then()`，它是**非同步**的，會等所有主程式跑完才回頭執行。|

### 4.4 如果沒有呼叫 resolve 或 reject 會發生什麼？

如果你的初始函式建立了一個 `new Promise`，但裡面的程式碼既沒有呼叫 `resolve()` 也沒有呼叫 `reject()`，那麼這個 Promise 的狀態會永遠卡在 **Pending（等待中）**。

```javascript
function silentPromise() {
    return new Promise((resolve, reject) => {
        console.log("任務開始，但我忘了寫 resolve...");
        // 這裡沒有 resolve() 也沒有 reject()
    });
}

console.log("準備執行...");

silentPromise()
    .then((data) => {
        // 這行永遠不會印出！
        console.log("收到結果：" + data); 
    })
    .catch((err) => {
        // 這行也永遠不會印出！
        console.log("抓到錯誤");
    });

console.log("主程式跑完了，但上面的 then 還在等...");
```

**後果：** 後面的 `.then()` 或 `.catch()` **永遠不會被執行**。

---

## 第五章：如何「使用」Promise 的結果？

既然 Promise 已經發出了「成功」或「失敗」的通知，接下來我們就要學習如何接收這些訊息。JavaScript 提供了三個非常重要的方法：

- **`.then()`**：當 Promise 成功時 (`resolve`)，會執行的動作。
- **`.catch()`**：當 Promise 失敗時 (`reject`)，會執行的動作。
- **`.finally()`**：不論成功或失敗都會執行的動作。

### 5.1 `.then()` 與 `.catch()` 的基本用法

```javascript
const weatherPromise = new Promise((resolve, reject) => {
    let weather = "下雨"; // 假設現在的狀況

    if (weather === "晴天") {
        resolve("陽光普照");
    } else {
        reject("下大雨了");
    }
});

// 開始使用這個 Promise
weatherPromise
    .then((data) => {
        // 如果成功 (resolve)，會跑這裡
        console.log("太棒了！結果是：" + data + "，我們去野餐吧！");
    })
    .catch((error) => {
        // 如果失敗 (reject)，會跑這裡
        console.log("喔不！原因如下：" + error + "，只能待在家了。");
    });
```

**背後的運作邏輯：**

1. **資料傳遞**：你在 `resolve("陽光普照")` 括號裡放的東西，會變成 `.then((data) => ...)` 裡面的 `data`。
2. **錯誤攔截**：如果你在 `reject("下大雨了")` 裡寫了原因，`.catch((error) => ...)` 就會幫你捉住這個 `error` 並處理它。

### 5.2 `.finally()` 的用途

`.finally()` 的主要目的是做「清理工作」（例如：不論連線成功或失敗，最後都要關閉載入中的視窗）。

**`.finally()` 的特殊之處：**

- 它**不接收任何參數**。
- 它通常**不會改變鏈條傳遞的資料**。

```javascript
Promise.resolve("☕ 拿鐵")
    .finally(() => {
        console.log("清理櫃檯...");
        return "🍰 蛋糕"; // 這裡回傳了東西
    })
    .then((data) => {
        console.log("最後拿到：" + data); // 印出「拿鐵」，不是「蛋糕」
    });
```

> **重點**：`finally` 就像是一個「透明的過路站」。它執行完內部的程式碼後，會自動把「進入 `finally()` 之前的結果」原封不動地交給下一棒。

---

## 第六章：實際應用範例 - 封裝 jQuery AJAX

在實務開發中，Promise 最常用於處理 AJAX 請求。以下是一個使用 jQuery 的範例：

```javascript
// 封裝 jQuery AJAX 為 Promise
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        $.ajax({
            url: `/api/users/${userId}`,
            method: 'GET',
            dataType: 'json'
        })
        .done((data) => {
            // AJAX 成功時解析 Promise
            resolve(data);
        })
        .fail((xhr, status, error) => {
            // AJAX 失敗時拒絕 Promise
            reject(new Error(`載入使用者資料失敗: ${error}`));
        });
    });
}

// 使用封裝好的函式
fetchUserData(123)
    .then((userData) => {
        console.log("使用者資料：", userData);
        // 更新 UI 顯示使用者資訊
        $('#userName').text(userData.name);
        $('#userEmail').text(userData.email);
    })
    .catch((error) => {
        console.error("載入失敗：", error);
        // 顯示錯誤訊息給使用者
        $('#errorMessage').text(error.message).show();
    });
```

### 6.1 簡潔寫法：直接傳遞函式引用

如果不需要額外加工資料，可以使用更簡潔的寫法：

```javascript
function fetchUserData(userId) {
    return new Promise((resolve, reject) => {
        $.ajax({ url: `/api/users/${userId}` })
            .done(resolve) // 成功直接轉給 resolve
            .fail(reject); // 失敗直接轉給 reject
    });
}
```

**為什麼可以這樣寫？**

在 JavaScript 中，函式可以像變數一樣被傳來傳去。當你寫 `.done(resolve)` 時，你是在告訴 jQuery：「如果成功了，請直接幫我執行 `resolve` 這個函式，並把你拿到的資料通通塞給它。」

---

## 第七章：完整練習範例

```html
<script type="text/javascript">
    
    function createPromise() {    
        return new Promise((resolve, reject) => {        
            var success = true; 
            setTimeout(function() {
                if(success)
                    resolve('資料處理完成!!');
                else
                    reject('資料處理失敗!!');
            }, 3000);
        });
    }

    function launchPromise() {
        
        console.log('launchPromise() 開始');
        
        // 秒數計數，是異步作業
        var procCount = 0;
        var procInterval = setInterval(() => {
            procCount++;
            console.log(`Promise處理中 (${procCount}秒)...`);
        }, 1000);
        
        createPromise()
            .then(result => {
                console.log('成功：', result);
                return '處理後的資料-xxx'; 
            })
            .then(newResult => {            
                console.log('進一步處理：', newResult);            
            })
            .catch(error => {
                console.error('錯誤：', error);            
            })
            .finally(() => {
                console.log('工作完成 (無論成功失敗都會執行)');
                // 清除(停止) progressInterval
                clearInterval(procInterval);
            });
    }
</script>

<input type="button" value="啟動 Promise" onclick="launchPromise()" style="margin: 10px;">
```

> **最佳實踐**：在 `finally` 裡面呼叫 `clearInterval` 是**教科書等級的標準做法**！因為不論成功或失敗都需要停止計數，寫在 `finally` 裡就不用在 `.then` 跟 `.catch` 裡面重複寫兩次。這就是 `finally` 存在的最大價值：**減少重複代碼 (DRY - Don't Repeat Yourself)**。

---

## 本章小結

在這一篇中，我們學習了：

1. **同步與非同步**的差異，以及為什麼需要 Promise。
2. **Promise 的三種狀態**：Pending、Fulfilled、Rejected。
3. **resolve/reject（動作）** 與 **fulfilled/rejected（狀態）** 的區別。
4. 如何**建立 Promise** 以及 Promise 內容會**立即執行**的特性。
5. 如何使用 **`.then()`、`.catch()`、`.finally()`** 處理結果。
6. 實際應用：**封裝 jQuery AJAX 為 Promise**。

在下一篇中，我們將深入探討 **Promise 鏈式調用 (Chaining)** 與 **回傳機制**，這是掌握 Promise 最核心的觀念！

---

[[JavaScript Promise 完整教學（二）：鏈式調用與回傳機制 | 下一篇：JavaScript Promise 完整教學（二）：鏈式調用與回傳機制]]
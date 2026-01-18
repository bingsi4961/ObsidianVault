---
date: 2026-01-18 15:20
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
#### 📑 [[JavaScript Promise 完整教學（一）：基礎觀念與核心機制]]
#### 📑 [[JavaScript Promise 完整教學（三）：進階技巧與並行處理]]

---

## 第一章：Promise 鏈式調用的基礎

### 1.1 為什麼可以一直 `.then().then().catch()` 寫下去？

這是因為 **`.then()`、`.catch()`、`.finally()` 都會回傳一個「全新的 Promise 物件」**。

如果 `.then()` 不回傳 Promise，它就只是一個普通的函式呼叫，鏈條就會在那裡斷掉。

```javascript
const p1 = Promise.resolve("起點");

const p2 = p1.then((data) => {
    return data + " -> 第一站";
});

console.log(p1 === p2); // 結果會是 false！
```

**為什麼要回傳「新」的 Promise？**

因為如果 `p1` 和 `p2` 是同一個物件，那當 `p2` 變成成功時，`p1` 的狀態也會跟著變，這會導致邏輯大亂。透過回傳**新的** Promise，每一段鏈條都可以擁有自己獨立的狀態和結果。

### 1.2 生活例子：接力賽

Promise 鏈就像是一個**接力賽**：

1. **第一棒 (原始 Promise)**：負責起跑（例如：去抓資料）。
2. **中繼站 (.then)**：
    - 它是一個全新的跑者。
    - 當第一棒把棒子交給它時，它開始跑自己的路段。
    - **不管它跑得如何，它最終都要產生一個「新的結果」交給下一棒。**

---

## 第二章：`.then()` 的回傳機制

在 Promise 的 `.then()` 回呼函數中，當您 `return` 一個值時，這個值會**自動被包裝成一個已解決（resolved）的 Promise**，然後傳遞給下一個 `.then()`。

### 2.1 四種回傳情境

|你在 `.then()` 回傳的東西|下一個 `.then()` 接收到的結果|狀態|
|---|---|---|
|**一個普通數值** (如 `return "OK"`)|下一個 `.then` 會立即收到 `"OK"`。|Fulfilled|
|**另一個 Promise** (如 `return getProfilePic()`)|下一個 `.then` 會「等待」這個 Promise 完成，才拿到結果。|取決於該 Promise|
|**拋出錯誤** (如 `throw new Error()`)|會直接跳過後面的 `.then`，進入 `.catch`。|Rejected|
|**沒有任何 return**|下一個 `.then` 會收到 `undefined`。|Fulfilled|

### 2.2 範例：觀察不同回傳值的流轉

```javascript
// 準備模擬函式
function checkAccount() {
    return new Promise((resolve) => {
        setTimeout(() => resolve("帳號已驗證"), 500);
    });
}

function getProfilePic() {
    return new Promise((resolve) => {
        setTimeout(() => resolve("https://image.jpg"), 500);
    });
}

// 觀察鏈式呼叫的流程
checkAccount()
    .then((msg) => {
        console.log("步驟 1:", msg);
        return "OK"; // (1) 回傳普通值
    })
    .then((prev) => {
        console.log("步驟 2 收到:", prev); // 收到 "OK"
        return getProfilePic(); // (2) 回傳另一個 Promise
    })
    .then((url) => {
        console.log("步驟 3 收到:", url); // 收到網址，而非 Promise 物件本身
        // (3) 故意不寫 return
    })
    .then((data) => {
        console.log("步驟 4 收到:", data); // 收到 undefined
        throw new Error("人為錯誤"); // (4) 拋出錯誤
    })
    .catch((err) => {
        console.log("捕捉到錯誤:", err.message);
    });
```

### 2.3 重要觀念：為什麼 `then` 會把 Promise「拆開」？

在 JavaScript 中，這叫做 **「Promise 展平」(Flattening)**。

當你在 `.then()` 裡面回傳一個 Promise 時，後面的 `.then()` **不會**收到那個 Promise 物件，而是會收到該 Promise **成功後的值**。

**原理：** `then` 方法的設計初衷就是為了處理「連續的任務」。如果它不幫你拆開，你就會得到「盒子裡還有盒子」的狀況，程式碼會變得極難處理。

**行為：** 當 `then` 發現你回傳的是一個 Promise，它會自動停下來，等待這個 Promise 變成 `Fulfilled`（成功），然後把結果取出來，交給下一個 `.then`。

### 2.4 沒有 return 會發生什麼事？

`.then()` 永遠會回傳一個**新的 Promise**。如果你在 `.then()` 的函式中沒有寫 `return`，JavaScript 會預設回傳 `undefined`。

```javascript
// 原始寫法
.then((result) => {
    console.log('處理中...');
    // 沒有 return 語句
})

// 等同於
.then((result) => {
    console.log('處理中...');
    return undefined;
})

// 進一步等同於
.then((result) => {
    console.log('處理中...');
    return Promise.resolve(undefined);
})
```

**重點：undefined 不會觸發 catch！**

當 `.then()` 中沒有 `return` 時：

- 下一個 `.then()` 會收到 `undefined`。
- 這仍然是一個**成功的 Promise**（Fulfilled 狀態）。
- 會繼續執行下一個 `.then()`。
- **不會**跳到 `.catch()`。

### 2.5 常見陷阱：`return new Error()` vs `throw new Error()`

很多人以為 `return new Error()` 會讓 Promise 變成失敗，但其實不會！

```javascript
// ⚠️ 這不會讓 Promise 變成失敗！
.then(() => {
    return new Error("這是一個錯誤物件"); // 只是回傳一個 Error 物件
})

// ✅ 這才會讓 Promise 變成失敗
.then(() => {
    throw new Error("這會觸發 catch"); // 拋出錯誤
})
```

在 JavaScript 中，`Error` 只是一個一般的「物件」。如果你希望 Promise 變成失敗狀態，你必須使用 **`throw`** 或者 **`return Promise.reject()`**。

---

## 第三章：`.catch()` 的回傳機制

`.catch()` 的本質是「錯誤處理」，就像車子壞了（Rejected），進了修車廠（`.catch`），只要維修完畢（正常 return），車子出來時就是好的（Fulfilled），所以會繼續跑下一個 `.then()`。

### 3.1 `.catch()` 後的狀態轉換

```javascript
// 情況 1：catch 有 return（修復成功）
function myFunction() {
    return Promise.resolve('開始')
        .then(result => {
            throw new Error('發生錯誤');
        })
        .catch(error => {
            console.log('捕獲錯誤:', error.message);
            return '錯誤已處理';  // 明確回傳值
        });
    
    // 🔥 整個鏈式回傳 Promise.resolve('錯誤已處理')
    // 注意是 resolve 喔!!
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
    
    // 🔥 整個鏈式回傳 Promise.resolve(undefined)
    // 注意是 resolve 喔!!
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

**重要觀念：**

- `.catch()` 也會回傳一個新的 Promise。
- 🔥 如果 `.catch()` 正常執行（不拋出錯誤），會回傳 **Resolved Promise**。
- 🔥 如果 `.catch()` 中又拋出錯誤，會回傳 **Rejected Promise**。

### 3.2 生活例子：醫院看病

1. 你的身體出了問題（**Rejected**）。
2. 你去看了醫生（進入了 **`.catch()`**）。
3. 醫生幫你把病治好了，你離開醫院時，身體是健康的（回傳 **Fulfilled**）。

---

## 第四章：`.finally()` 的回傳機制

`.finally()` 的行為與 `.then()` 和 `.catch()` 完全不同，它被設計為一個「不干涉」的區塊。它會盡力讓前一棒的結果「直接通過」。

### 4.1 `.finally()` 需要帶參數嗎？

**答案是：不需要，也不應該帶參數。**

在 JavaScript 的規範中，傳遞給 `finally()` 的回呼函式（Callback function）是**不接受任何參數**的。

- **`.then(value => ...)`**：會接收到成功的「值」。
- **`.catch(error => ...)`**：會接收到失敗的「原因」。
- **`.finally(() => ...)`**：什麼都不會收到。

### 4.2 `then()` 或 `catch()` 的值會傳進 `finally()` 嗎？

**不會。** `finally` 的設計初衷是為了「清理現場」，它並不關心前面到底是成功還是失敗，也不關心具體的資料是什麼。

這就像是你在實驗室做實驗 🧪：

1. **實驗成功**（`.then`）：你拿到了實驗數據。
2. **實驗失敗**（`.catch`）：你記錄了失敗的原因。
3. **不論成功或失敗**（`.finally`）：你最後都要**把實驗袍掛好、把燈關掉**。

清理（關燈）這個動作，並不需要知道實驗數據是什麼，也不需要知道失敗原因是什麼。

### 4.3 三個方法的參數接收比較表

|方法|接收的參數內容|會收到前面 return 的值嗎？|主要用途|
|---|---|---|---|
|**`.then()`**|`result` (成功的值)|**會**，收到前一棒的 return|處理資料、執行下一步任務。|
|**`.catch()`**|`error` (失敗原因)|**會**，收到前一棒拋出的錯誤|捕捉錯誤、進行補救。|
|**`.finally()`**|**無** (空參數)|**不會**，它會「忽略」並「傳遞」值|清除定時器、關閉 Loading 圖示。|

### 4.4 測驗：如果在 `finally` 中寫了參數會怎樣？

```javascript
somePromise
    .then(result => result)
    .finally((data) => { 
        console.log(data); // 印出什麼？
    });
```

**答案：`undefined`**

因為 `finally` 的設計就是「不參與資料流」，所以它不會接收到任何結果，即使你寫了參數，該參數的值也會是 `undefined`。

### 4.5 `.finally()` 的特性

|程式碼情境|在 `finally()` 中的結果|下一步去哪？|
|---|---|---|
|**`return "資料"`**|**維持前一棒狀態** (忽略此 return)|回到前一棒該去的地方 (傳遞前一棒的值)|
|**沒有任何 `return`**|**維持前一棒狀態**|回到前一棒該去的地方|
|**`throw new Error()`**|**Rejected** (失敗) ❌|**強行跳到** 下一個 `.catch()`|
|**`return Promise.reject()`**|**Rejected** (失敗) ❌|**強行跳到** 下一個 `.catch()`|

### 4.6 範例：finally 不改變傳遞的資料

```javascript
// 情況 1：finally 有 return（通常被忽略）
Promise.resolve('成功結果')
    .then(result => {
        console.log('處理:', result);
        return result;
    })
    .finally(() => {
        console.log('清理工作');
        return '這個回傳值會被忽略';  // 這個 return 不會影響結果
    })
    .then(data => {
        console.log('最後拿到:', data); // 印出「成功結果」，不是「這個回傳值會被忽略」
    });
```

### 4.7 為什麼 `finally()` 裡的 `return` 會被忽略？

因為 `finally()` 的功能是「打掃」。不論任務成功與否，都要關燈、關冷氣。如果你在打掃時突然說「拿這塊蛋糕去給下一棒」（`return "蛋糕"`），系統會覺得這不是你的職責，所以會直接把原本就在傳遞的東西繼續傳下去。

### 4.8 程式碼觀察：為什麼值會「跳過」`finally`？

如果在 `finally` 之後再接一個 `.then()`，你會發現資料依然存在：

```javascript
createPromise()
    .then(result => {
        return '處理後的資料-xxx'; 
    })
    .finally(() => {
        console.log('清理中...'); // 這裡沒辦法拿到 '處理後的資料-xxx'
    })
    .then(data => {
        console.log('最後一棒拿到：', data); // 這裡依然能拿到 '處理後的資料-xxx'
    });
```

這證實了我們之前說的：`finally` 是一個 **「透明的過路站」**。它執行完內部的程式碼後，會自動把「進入 `finally` 之前的結果」原封不動地交給下一棒。

### 4.9 唯一例外：在 finally 中拋出錯誤

```javascript
Promise.resolve('成功結果')
    .finally(() => {
        throw new Error('finally 中的錯誤');
    })
    .then(data => {
        console.log('這行不會執行');
    })
    .catch(err => {
        console.log('抓到錯誤:', err.message); // 印出「finally 中的錯誤」
    });
```

---

## 第五章：綜合比較表

### 5.1 三種方法的回傳機制總覽

|方法|需要手動 `resolve/reject`？|自動處理邏輯|
|---|---|---|
|**`new Promise`**|**是** (必備)|它只是一個空殼，你必須親自呼叫通知官。|
|**`.then()`**|**否** (自動)|`return` 值 → Resolve；`throw` 錯誤 → Reject|
|**`.catch()`**|**否** (自動)|`return` 值 → Resolve (修復成功)；`throw` 錯誤 → Reject|
|**`.finally()`**|**否** (自動)|**特殊：** 它不看 `return` 值，會直接把前一棒的結果「傳遞」下去。只有在它出錯時才會變 Reject。|

### 5.2 完整情境對照表

|程式碼情境|在 `.then()` 中的結果|在 `.catch()` 中的結果|下一步去哪？|
|---|---|---|---|
|**`return "資料"`** (純數值)|**Fulfilled** (成功)|**Fulfilled** (成功)|下一個 `.then()`|
|**`return new Error()`**|**Fulfilled** (成功) ⚠️|**Fulfilled** (成功) ⚠️|下一個 `.then()` (收到的是 Error 物件)|
|**沒有任何 `return`**|**Fulfilled** (成功)|**Fulfilled** (成功)|下一個 `.then()` (收到 `undefined`)|
|**`throw new Error()`**|**Rejected** (失敗)|**Rejected** (失敗)|下一個 `.catch()`|
|**`return Promise.reject()`**|**Rejected** (失敗)|**Rejected** (失敗)|下一個 `.catch()`|

---

## 第六章：Promise 鏈的實戰應用

### 6.1 多步驟的登入流程

```javascript
function loginUser(credentials) {
    return new Promise((resolve, reject) => {
        // 模擬登入 API 呼叫
        setTimeout(() => {
            if (credentials.username && credentials.password) {
                resolve({ token: "abc123", userId: 456 });
            } else {
                reject(new Error("登入資訊不完整"));
            }
        }, 1000);
    });
}

function fetchUserProfile(token, userId) {
    return new Promise((resolve, reject) => {
        // 模擬取得使用者資料的 API 呼叫
        setTimeout(() => {
            resolve({
                id: userId,
                name: "張小明",
                department: "IT部門"
            });
        }, 500);
    });
}

// 鏈接多個 Promise
loginUser({ username: "user", password: "pass" })
    .then((loginResult) => {
        console.log("登入成功，Token:", loginResult.token);
        // 回傳下一個 Promise 給下一個 .then()
        return fetchUserProfile(loginResult.token, loginResult.userId);
    })
    // Promise 鏈自動解析結果給 userProfile
    .then((userProfile) => {
        // userProfile 不是 Promise 物件，而是 { id: userId, name: "張小明", department: "IT部門" }
        console.log("使用者資料載入完成：", userProfile);
        // 更新頁面顯示
        updateUserInterface(userProfile);
    })
    .catch((error) => {
        // 🔥 會捕捉到整個 Promise 鏈中任何一個環節發生的錯誤。
        console.error("操作過程發生錯誤：", error);
        showErrorMessage(error.message);
    });
```

### 6.2 接力賽的失敗處理

如果其中一棒跑者跌倒了（`reject`），後面的跑者就不會再接棒，比賽會直接終止，並由醫療人員（`.catch`）接手處理。

```javascript
checkAccount()
    .then((msg) => {
        console.log(msg); // 這裡印出「帳號 OK」
        return getProfilePic(); // 接著執行第二個任務
    })
    .then((url) => {
        console.log("拿到圖片了：" + url);
    })
    .catch((err) => {
        console.log("發生錯誤：" + err);
    });
```

**如果 `checkAccount`（第一步）就失敗了，程式還會去執行 `getProfilePic`（第二步）嗎？**

**答案：不會。** 失敗狀態會順著鏈條往下傳，直到遇到 `.catch()` 為止。

---

## 第七章：在 `.then()` 中使用 `$.ajax`

### 7.1 關鍵：有沒有寫 `return`

|情況|寫法範例|會發生什麼事？|會被之後的 `.catch` 捕捉嗎？|
|---|---|---|---|
|**有 return**|`return $.ajax(...)`|鏈條會**等待**網路請求完成，才跑下一棒。|**會！** 網路出錯會跳到 `.catch`。|
|**沒 return**|`$.ajax(...)`|鏈條**不會等待**，直接跑下一棒。|**不會！** 請求失敗時，Promise 鏈已經跑完了。|

**實作範例：當 `$.ajax` 發生錯誤時**

```javascript
checkAccount()
  .then((data) => {
    console.log("帳號檢查通過，準備抓取資料...");
    
    // 情境：我們回傳一個 $.ajax 請求
    return $.ajax({
      url: "https://api.example.com/user/123",
      method: "GET"
    });
  })
  .then((userInfo) => {
    // 只有在 ajax 成功時才會執行到這裡
    console.log("抓到資料了：", userInfo);
  })
  .catch((error) => {
    // 只要 checkAccount 失敗，或者 $.ajax 失敗（如 404 或網路斷線）
    // 都會統一跳到這裡處理！
    console.log("流程中發生錯誤：", error.statusText);
  });
```

### 7.2 為什麼「沒寫 return」會導致 catch 不到？

如果你沒寫 `return`，對 Promise 鏈來說，你在這個 `.then` 裡面只是啟動了一個「背景任務」，然後就「結束並回傳了 undefined」。

就像你叫店員去煮咖啡，但他轉身去啟動機器後就立刻回頭跟你說「我做完了」（回傳了 undefined），而不是「等咖啡煮好再回報」。這時候機器就算爆炸了（ajax 失敗），店員也已經宣稱他任務完成，`.catch` 自然接不到這個錯誤。

### 7.3 深入釐清：沒有 return 對 catch 的兩種情況

「沒寫 return 會導致 catch 不到」這句話需要拆解成兩個層面來理解：

|情況|發生了什麼事？|`catch` 能抓到嗎？|
|---|---|---|
|**情況 A：Promise 內部出錯**|在 `then` 裡面直接寫 `throw new Error()`|**能！** 因為這是在這一段執行時發生的。|
|**情況 B：非同步任務（如 AJAX）失敗**|沒寫 `return`，但 `$.ajax` 失敗了|**不能！** 因為 `then` 已經結束了，AJAX 的錯誤發生在「未來」，主鍊條已經抓不到它了。|

> **重點整理：** 沒寫 `return`，Promise 鏈就會「斷掉」對那個非同步任務的監控。對鏈條來說，回傳值就是 `undefined`（成功狀態），所以它會繼續往下跑 `.then`，而不會進到 `.catch`。

### 7.4 測驗解析：沒寫 return 時，下一個 then 何時執行？

請看這段程式碼：

```javascript
promiseA()
  .then(() => {
    $.ajax({ url: "/api/test" }); // 注意：這裡沒寫 return
  })
  .then(() => {
    console.log("任務完成！");
  });
```

**問題：如果 `/api/test` 的網路請求需要跑 10 秒鐘，那「任務完成！」這行字是在「10 秒後」印出，還是「幾乎立刻」印出呢？**

**答案：「幾乎立刻」**

因為 JavaScript 的 `.then()` 就像是一個**快遞員**：

- 如果你給他一個 `return`，他會乖乖拿著東西送到下一站。
- 如果你給他一個 **Promise**（加上 `return`），他會在那裡等包裹拆開，再送去下一站。
- **但是**，如果你沒寫 `return`，快遞員跑進房間（執行了 `$.ajax`），看到裡面沒東西要給他，他會以為任務結束了，直接兩手空空（帶著 `undefined`）衝向下一站。

所以，雖然 `$.ajax` 在背景慢慢跑 10 秒，但 `.then()` 早就不等它，直接執行下一個 `then` 印出「任務完成！」了。

### 7.5 箭頭函式的隱含回傳

要特別注意**括號**的使用：

- **自動回傳：** `() => $.ajax(...)` （沒有花括號 `{}`）。這會自動 `return` 該 AJAX 物件。下一棒會**等待**。
- **手動回傳：** `() => { $.ajax(...) }` （有花括號 `{}`）。這**不會**自動回傳。下一棒**不會等待**，且會收到 `undefined`。

---

## 第八章：鏈式中的最後一個處理器

無論鏈式中的最後一個 `.then()` 有沒有 `return`，整個 Promise 鏈都會回傳一個 Promise 給呼叫的 function。

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

- `.then()` 永遠回傳一個新的 Promise。
- 鏈式中的最後一個 `.then()` 決定整個鏈式的最終回傳值。
- 無論有沒有明確 `return`，都會回傳 Promise 給呼叫的 function。

---

## 本章小結

在這一篇中，我們深入學習了：

1. **Promise 鏈式調用**的運作原理：每個 `.then()`、`.catch()`、`.finally()` 都會回傳一個新的 Promise。
2. **`.then()` 的回傳機制**：四種情境（回傳普通值、回傳 Promise、拋出錯誤、沒有 return）。
3. **Promise 展平 (Flattening)**：`then` 會自動把回傳的 Promise「拆開」。
4. **`.catch()` 的回傳機制**：正常執行會回傳 Resolved Promise（修復成功）。
5. **`.finally()` 的回傳機制**：透明過路站，不改變傳遞的資料。
6. **實戰應用**：多步驟登入流程與 `$.ajax` 整合。
7. **常見陷阱**：`return new Error()` vs `throw new Error()`、箭頭函式的隱含回傳。

在下一篇中，我們將學習如何**同時處理多個 Promise**（`Promise.all`、`Promise.allSettled`、`$.when`），以及 **`Promise.resolve()`** 與 **`Promise.reject()`** 的快捷用法！

---

[[JavaScript Promise 完整教學（一）：基礎觀念與核心機制 | 上一篇：JavaScript Promise 完整教學（一）：基礎觀念與核心機制]]
[[JavaScript Promise 完整教學（三）：進階技巧與並行處理 | 下一篇：JavaScript Promise 完整教學（三）：進階技巧與並行處理]]
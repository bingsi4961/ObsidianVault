---
date: 2025-07-07 22:15
aliases: 
tags:
  - JavaScript
  - async/await
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
##### 📑 [[JavaScript Promise 教學]]
##### 📑 [[Promise async await 範例]]

---

## 從你熟悉的 C# 開始理解

在 C# 中，你可能已經寫過這樣的程式碼：

```csharp
public async Task<string> GetUserDataAsync(int userId)
{
    var response = await httpClient.GetAsync($"/api/users/{userId}");
    var content = await response.Content.ReadAsStringAsync();
    return content;
}
```

JavaScript 的 async/await 概念幾乎一模一樣，只是語法稍有不同。這種相似性會讓你的學習過程更加順暢。

## 理解 async/await 的核心概念

async/await 實際上是 Promise 的一種更優雅的寫法。當你在函式前面加上 `async` 關鍵字，這個函式就會==自動回傳一個 Promise==。而 `await` 關鍵字則讓你可以「等待」一個 Promise 完成，就像暫停程式執行直到結果出現。

讓我們先看一個簡單的對比，幫助你理解兩者之間的關係：

```javascript
// 使用傳統 Promise 的寫法
function fetchUserDataWithPromise(userId) {
    return fetch(`/api/users/${userId}`)
		🔥🔥🔥
		// then() 告訴 JavaScript「等 fetch 完成後，再執行我這個函式」
		// 因為 fetch 完成，then() 可以從 Promise<Response> 直接取得 Response
		
		// 取得回應內容：從 Response 物件中讀取 HTTP 回應的 body 內容
	    // 解析 JSON：將回應內容（通常是 JSON 字串）解析成 JavaScript 物件
	    // 回傳 Promise：這是一個非同步操作，所以回傳 Promise
        .then(response => response.json())
        .then(userData => {
            console.log("取得使用者資料：", userData);
            return userData; // return Promise.resolve(userData);
        })
        .catch(error => {
            console.error("發生錯誤：", error);
            throw error; // return Promise.reject(error);
            // 🔥🔥 若沒有寫 throw error，則系統認為您已將錯誤理處好了，自動 return Promise.resolve(undefined)
        });
}

// 使用 async/await 的寫法 - 是不是看起來更清楚？
async function fetchUserDataWithAsync(userId) {
    try {
	    🔥🔥🔥
	    // await 是用來「等待」Promise 完成
		// 如果不加 await，程式會立即繼續執行下一行，不會等待結果
	    // fetch 完成後回傳 Promise<Response>，然後透過 await 取得 Response
	    // 之前學過，有加 await，response 會是實際資料 (因為 fetch 已完成)
	    // 若沒有加 await，fetch 可能未完成，所以 response 是 Promise<Response>
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        console.log("取得使用者資料：", userData);
        return userData; // return Promise.resolve(userData);
	    // 如果沒有 return userData; 則自動 return Promise.resolve(undefined);
    } catch (error) {
        console.error("發生錯誤：", error);
        throw error; // return Promise.reject(error)
        // 🔥🔥 若沒有寫 throw error，則系統認為您已將錯誤理處好了，自動 return Promise.resolve(undefined)
    }
}
```

你注意到了嗎？async/await 版本讀起來就像是一般的同步程式碼，但實際上它仍然是異步執行的。這種可讀性的提升是 async/await 最大的優勢。

```javascript
fetch() 
    ↓
Promise<Response>  ← 這是一個「承諾」，不是實際的 Response
    ↓ (需要等待 await 或 then)
Response           ← 這才是真正的 Response 物件
    ↓ (.json() 方法)
Promise<Object>    ← response.json() 也回傳 Promise
    ↓ (再次等待)
Object             ← 最終的 JSON 物件
```

```javascript
● fetchUserDataWithAsync(123) 取得 Promise<Object>
● await fetchUserDataWithAsync(123)  取得 Object

● 若 fetchUserDataWithAsync(123) 回傳 Promise<Error> 時
● 🔥await fetchUserDataWithAsync(123) 會拋出例外，因為已可從 Promise.reject(error) 中取得 error，外部的 try/catch 會捕捉到例外
● 🔥但若 fetchUserDataWithAsync(123) 的話，就會回傳 Promise<Error>，但不會發生例外 
```

#### 處理未使用 await 的 rejected Promise

```javascript
// 方式 1：使用 .catch()
var dd = fetchUserDataWithAsync(123);
dd.catch(error => {
    console.log("處理錯誤：", error.message);
});

// 方式 2：使用 .then().catch()
var dd = fetchUserDataWithAsync(123);
dd.then(data => {
    console.log("成功：", data);
}).catch(error => {
    console.log("錯誤：", error.message);
});
```

## async 函式的特性

當你在函式前面加上 `async` 關鍵字時，這個函式會有幾個重要的特性：
首先，==它總是回傳一個 Promise==。即使你在函式中直接回傳一個值，JavaScript 也會自動將它包裝成一個 resolved 的 Promise。

```javascript
// 這個函式會回傳 Promise<string>
async function greetUser() {
    return "你好！"; // 自動包裝成 Promise.resolve("你好！")
}

// 所以你可以這樣使用它
greetUser().then(message => console.log(message)); // 輸出：你好！
```

其次，==只有在 async 函式內部才能使用 `await` 關鍵字==。這是一個重要的規則，記住它會幫助你避免很多錯誤。

## await 的運作原理

`await` 關鍵字的作用是暫停函式的執行，直到 Promise 解析完成。重要的是要理解，這種「暫停」不會阻塞整個程式，只是暫停當前的函式執行流程。

```javascript
async function demonstrateAwait() {
    console.log("開始執行");
    
    // 這裡會暫停，等待 Promise 完成
    const result = await new Promise(resolve => {
        setTimeout(() => resolve("延遲的結果"), 2000);
    });
    
    console.log("取得結果：", result);
    console.log("函式執行完成");
}

// 呼叫函式
demonstrateAwait();
console.log("這行會立即執行，不會等待上面的函式完成");
```

這個範例會輸出：

```
開始執行
這行會立即執行，不會等待上面的函式完成
（等待 2 秒）
取得結果：延遲的結果
函式執行完成
```

## 在你的工作環境中應用 async/await

讓我們看看如何在你的 Portal 系統（使用 jQuery 3.3.1）中應用這些概念。
jQuery 本身並不直接支援 Promise，但我們可以輕鬆地將 jQuery AJAX 轉換為 Promise：

```javascript
// 建立一個輔助函式，將 jQuery AJAX 轉換為 Promise
function ajaxPromise(options) {
    return new Promise((resolve, reject) => {
        $.ajax(options)
            .done(resolve)  // 成功時解析 Promise
            // .done((data, textStatus, jqXHR) => { resolve(data, textStatus, jqXHR); })
            .fail(reject);  // 失敗時拒絕 Promise
            // .fail((jqXHR, textStatus, errorThrown) => { reject(jqXHR, textStatus, errorThrown); })
    });
}

// 現在你可以在 Portal 系統中使用 async/await
async function loadUserProfile(userId) {
    try {
        // 載入基本使用者資料
        const userData = await ajaxPromise({
            url: `/api/users/${userId}`,
            method: 'GET',
            dataType: 'json'
        });
        
        // 根據使用者資料載入部門資訊
        const departmentData = await ajaxPromise({
            url: `/api/departments/${userData.departmentId}`,
            method: 'GET',
            dataType: 'json'
        });
        
        // 更新頁面顯示
        $('#userName').text(userData.name);
        $('#userEmail').text(userData.email);
        $('#userDepartment').text(departmentData.name);
        
        console.log("使用者資料載入完成");
        return { user: userData, department: departmentData };
        
    } catch (error) {
        console.error("載入使用者資料時發生錯誤：", error);
        $('#errorMessage').text('載入使用者資料失敗，請稍後再試').show();
        // 重新拋出錯誤讓呼叫者處理
        throw error; // 自動 return Promise.reject(error);
    }
}
```

我們先載入使用者資料，然後根據使用者的部門ID載入部門資訊，最後更新頁面。每一步都會等待前一步完成，但整個過程仍然是異步的。

## 錯誤處理：try/catch 的威力

async/await 的另一個巨大優勢是錯誤處理變得更加直觀。==與其使用 `.catch()`，你可以使用熟悉的 `try/catch` 區塊==：

```javascript
async function robustApiCall(endpoint, data) {
    try {
        // 第一步：驗證參數
        if (!endpoint) {
            throw new Error("API 端點不能為空");
        }
        
        // 第二步：發送請求
        // fetch 回傳 Promise<Response>
        const response = await fetch(endpoint, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${getAuthToken()}`
            },
            body: JSON.stringify(data)
        });
        
        // 第三步：檢查 HTTP 狀態
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        // 第四步：解析回應
        const result = await response.json();
        
        // 第五步：驗證回應格式
        if (!result.success) {
            throw new Error(result.message || "API 回應指示操作失敗");
        }
        
        return result.data;
        
    } catch (error) {
        // 所有步驟中的任何錯誤都會在這裡被捕獲
        console.error("API 呼叫失敗：", error.message);
        
        // 你可以根據不同的錯誤類型做不同的處理
        if (error.message.includes("HTTP 401")) {
            // 認證失敗，可能需要重新登入
            redirectToLogin();
        } else if (error.message.includes("HTTP 500")) {
            // 伺服器錯誤
            showServerErrorMessage();
        }
        
        throw error; // 重新拋出讓上層處理
    }
}
```

```javascript
async function safeDataFetch(url) { 
	try { 
		const response = await fetch('https://jsonplaceholder.typicode.com/posts'); 
		
		try {
			// 先嘗試解析為 JSON
			const jsonData = await response.json(); 
			return { type: 'json', data: jsonData }; 
		} catch (jsonError) { 
			// 如果 JSON 解析失敗,改用 text
			const textData = await response.text(); 
			return { type: 'text', data: textData };
			// 如果內部 catch 發生錯誤，會把錯誤丟給外部 catch
			// 🚨 注意這裡不是 return Promise.reject(error) 
			// 🚨 因為只有在「整個函式執行完畢」時，系統才會決定返回 Promise.resolve 或 Promise.reject
		} 
	} catch (error) { 
		console.log('外部cath開始');
		throw new Error(`載入失敗: ${error.message}`); 
		// 🚨 這裡才會 return Promise.reject(new Error(`載入失敗: ${error.message}`));
	} 
}
```

## 並行處理：當你不需要等待時

有時候你有多個獨立的異步操作，不需要按順序執行。 這時候你可以結合 `Promise.all` 和 async/await：
#### 📑 [[Promise 中 resolve reject 與 catch 觸發機制筆記]]
#### 📑 [[$.ajax() 與 await 運作機制筆記]]

```javascript
/*
function ajaxPromise(options) {
	return new Promise((resolve, reject) => {			
		$.ajax(options)
			.done((result) => {
				resolve(result);
			})
			.fail((jqXHR) => {
				const errorMessage = `Ajax Error: ${jqXHR.status} ${jqXHR.statusText}`;
				const error = new Error(errorMessage);
				error.jqXHR = jqXHR;
				reject(error);
			});
	});
}
*/

// 這種寫法比較簡單
async function ajaxPromise(options) {
	try {
		// ajax() 回傳 deferred.resolve(result) 時，透過 await 取得 result
		var result = await $.ajax(options); 
		// return Promise.resolve(result)
		return result;
	} catch (jqXHR) {
		// ajax() 回傳 deferred.reject(jqXHR) 時觸發
		const errorMessage = `Ajax Error: ${jqXHR.status} ${jqXHR.statusText}`;
		const error = new Error(errorMessage);
		error.jqXHR = jqXHR;
		throw error;
	}
}

async function loadDashboardData(userId) {
	try {
		// 這些資料可以同時載入，因為它們互不依賴
		const [posts, users, post] = await Promise.all([
			ajaxPromise({ url: 'https://jsonplaceholder.typicode.com/posts', method: 'GET' }),
			ajaxPromise({ url: 'https://jsonplaceholder.typicode.com/users', method: 'GET' }),
			ajaxPromise({ url: `https://jsonplaceholder.typicode.com/posts/${userId}`, method: 'GET' })
		]);
		
		console.log(posts);
		console.log(users);
		console.log(post);			
		console.log('儀表板所有資料載入完成');
		
	} catch (error) {
		console.log('載入儀表板資料失敗：', error);
		throw error;	// return Promise.reject(error);
	}
}

async function launchLoadDashboardData() {		
	try {
		await loadDashboardData(1);
		🔥🔥🔥
		// 如果沒有加 await，就不會等待 loadDashboardData()，直接完成函式
		// 等 loadDashboardData() 完成後，外部也捕捉不到例外了			
		// 關鍵概念：try-catch 只能捕獲同步錯誤，或是有 await 的非同步錯誤
		
		// 沒有 await，就是沒有接收 loadDashboardData()
		// 真正的「接收」 => 等待 Promise 完成，如果成功，取得結果；如果失敗，拋出錯誤給 catch
		// await 就像 「我等你做完，告訴我結果」
		// 沒有 await 就像 「我知道你在做事，但我不等你」，我要去做後面的事了						
	} catch (error) {
		console.log('外部 Catch 觸發!!', error);			
	}		
}

launchLoadDashboardData();
```

## 避免常見的陷阱

讓我提醒你一些使用 async/await 時常見的錯誤，這樣你就能避免它們：

第一個常見錯誤是忘記在頂層使用 await。記住，你不能在非 async 函式中直接使用 await：

```javascript
// 錯誤的寫法
function normalFunction() {
    const result = await someAsyncOperation(); // 這會導致語法錯誤
}

// 正確的寫法
async function asyncFunction() {
    const result = await someAsyncOperation(); // 這是正確的
}

// 或者在頂層程式碼中，你需要這樣處理
someAsyncOperation().then(result => {
    console.log(result);
});
```

第二個常見錯誤是在迴圈中錯誤使用 await：

```javascript
// 這會依序執行，可能比你想的要慢
async function slowWay(userIds) {
    const results = [];
    for (const userId of userIds) {
        const userData = await fetchUserData(userId); // 每次都要等待
        results.push(userData);
    }
    return results;
}

// 這會並行執行，通常更快
async function fastWay(userIds) {
    const promises = userIds.map(userId => fetchUserData(userId));
    const results = await Promise.all(promises);
    return results;
}
```
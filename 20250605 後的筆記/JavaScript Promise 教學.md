---
date: 2025-07-05 22:57
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

#### 📑 [[Promise.then() 鏈中的 return 機制]]
#### 📑 [[鏈式回傳 Promise]]


JavaScript 的 Promise 是現代網頁開發中處理非同步操作的核心概念，我來為你詳細解說這個重要的主題。

## 什麼是 Promise？

Promise 就像是一個「承諾」或「約定」，它代表一個異步操作的最終完成或失敗。想像一下你在餐廳點餐，服務生給你一個號碼牌，這個號碼牌就像是一個 Promise - 它承諾你的餐點會準備好，但不知道確切的時間，也不確定是否會成功完成。

Promise 特別重要，因為它們可以幫助你更優雅地處理 AJAX 請求、檔案讀取等異步操作，而不會陷入「回調地獄」的困境。

## 什麼是 Promise？

Promise 就像是一個「承諾」或「約定」，它代表一個異步操作的最終完成或失敗。想像一下你在餐廳點餐，服務生給你一個號碼牌，這個號碼牌就像是一個 Promise - 它承諾你的餐點會準備好，但不知道確切的時間，也不確定是否會成功完成。

Promise 特別重要，因為它們可以幫助你更優雅地處理 AJAX 請求、檔案讀取等異步操作，而不會陷入「回調地獄」的困境。

## Promise 的三種狀態

Promise 有三種可能的狀態，就像一個任務的進行狀態：

**Pending（等待中）**：初始狀態，操作尚未完成也未失敗。就像你剛點完餐，廚師還在準備。

**Fulfilled（已實現）**：操作成功完成，當呼叫 resolve() 時，Promise 變成這個狀態。餐點準備好了，你可以享用。

**Rejected（已拒絕）**：操作失敗，當呼叫 reject() 時，Promise 變成這個狀態。可能廚房沒有食材，無法完成你的訂單。

重要的是，Promise 一旦從 pending 狀態轉換到 fulfilled 或 rejected，就無法再改變狀態。

## Promise 的執行機制

這是理解 Promise 運作的關鍵：**Promise 的 .then() 方法只有在 Promise 被 resolve 時才會執行，.catch() 方法只有在 Promise 被 reject 時才會執行。**

### 執行流程的關鍵

當你建立一個 Promise 時，裡面的程式碼會立即執行，但是：

- 如果沒有呼叫 resolve() 或 reject()，Promise 會永遠停留在 Pending 狀態
- 只有當 resolve() 被呼叫時，.then() 才會執行
- 只有當 reject() 被呼叫時，.catch() 才會執行
- .finally() 無論如何都會執行（只要 Promise 不是 Pending 狀態）

讓我用一個簡單的例子來示範：

javascript

```javascript
// 這個 Promise 會永遠卡在 Pending 狀態
function neverResolvePromise() {
    return new Promise((resolve, reject) => {
        console.log('Promise 內部程式碼執行了！');
        // 注意：這裡沒有呼叫 resolve() 或 reject()
        // 所以 Promise 會永遠停留在 Pending 狀態
    });
}

neverResolvePromise()
    .then(() => {
        console.log('這行永遠不會執行'); // 永遠不會看到這個
    })
    .catch(() => {
        console.log('這行也永遠不會執行'); // 永遠不會看到這個
    });

// 你只會在 console 看到：'Promise 內部程式碼執行了！'
```

這個例子清楚地展示了 Promise 必須透過 resolve() 或 reject() 來改變狀態，才能觸發後續的 .then() 或 .catch() 方法。

## 類比 C# 的 Task

這個概念其實和你熟悉的 C# Task 很相似。在 C# 中：

csharp

```csharp
// 類似的概念
var task = Task.Run(() => {
    // 如果這個方法沒有 return 或 throw exception
    // 呼叫者的 await 就會一直等待
});

// 只有當 task 完成時，await 後面的程式碼才會執行
await task;
Console.WriteLine("Task 完成了");
```

JavaScript 的 Promise 就像 C# 的 Task，必須有明確的「完成」信號（resolve/reject），後續的處理（.then/.catch）才會被觸發。

## 基礎語法與建立 Promise

讓我們從最基本的 Promise 建立開始：

```javascript
// 建立一個基本的 Promise
// ★★★ 建立 Promise 後，就立即執行內容了
const myPromise = new Promise((resolve, reject) => {
    // 這裡放你的異步操作
    const success = true; // 假設這是某種條件判斷
    
    if (success) {
	    // 成功時呼叫 resolve
	    // resolve 也是函式
        resolve("操作成功完成！");
    } else {
        // 失敗時呼叫 reject
        // reject 也是函式
        reject("操作失敗！");
    }
});
```

Promise 的建構函式接受一個執行函式（executor function），這個函式有兩個參數：`resolve` 和 `reject`。當異步操作成功時，你呼叫 `resolve`；當失敗時，你呼叫 `reject`。

## 使用 Promise - then 和 catch

建立 Promise 後，我們需要處理它的結果：

```javascript
myPromise
    .then((result) => {
        // 當 Promise 被 resolve 時執行
        console.log("成功：", result);
        return "處理後的資料"; // 可以回傳新的值給下一個 then
    })
    .then((newResult) => {
        // 可以鏈接多個 then
        console.log("進一步處理：", newResult);
    })
    .catch((error) => {
        // 當 Promise 被 reject 時執行
        console.log("錯誤：", error);
    })
    .finally(() => {
        // 無論成功或失敗都會執行
        console.log("清理工作完成");
    });
```

## 實際應用範例 - AJAX 請求

在你的工作環境中，Promise 最常用於處理 AJAX 請求。以下是一個使用 jQuery 的範例，你可以在 Portal 系統（jQuery 3.3.1）中使用：

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

## Promise 的鏈接 (Chaining)

Promise 的強大之處在於可以鏈接多個異步操作：

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
        // ★★ 會捕捉到整個 Promise 鏈中任何一個環節發生的錯誤。
        console.error("操作過程發生錯誤：", error);
        showErrorMessage(error.message);
    });
```

## Promise.all - 同時處理多個 Promise

當你需要同時執行多個異步操作並等待全部完成時，可以使用 `Promise.all`：

```javascript
// 假設需要同時載入多種資料
const loadDashboardData = () => {
    const userDataPromise = fetchUserData(123);
    const statisticsPromise = fetchStatistics();
    const notificationsPromise = fetchNotifications();
    
    return Promise.all([userDataPromise, statisticsPromise, notificationsPromise])
        .then((results) => {
            const [userData, statistics, notifications] = results;
            
            // 所有資料都載入完成，現在可以更新 UI
            updateUserSection(userData);
            updateStatisticsSection(statistics);
            updateNotificationsSection(notifications);
            
            console.log("儀表板資料全部載入完成");
        })
        .catch((error) => {
            // 任何一個 Promise 失敗都會觸發這裡
            console.error("載入儀表板資料時發生錯誤：", error);
        });
};
```

## Promise.race - 競速執行

有時候你可能想要「誰先完成就用誰的結果」，這時可以使用 `Promise.race`：

```javascript
// 設定逾時機制的範例
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
    .then(response => response.json())  // response => { return response.json(); }
    .then(data => console.log("資料載入成功：", data))
    .catch(error => console.error("載入失敗或逾時：", error));
```

## 錯誤處理的最佳實務

在你的系統開發中，良好的錯誤處理是關鍵：

```javascript
// 建議的錯誤處理模式
function robustDataFetch(endpoint) {
    return fetch(endpoint)
        .then(response => {
            // 檢查 HTTP 狀態碼
            if (!response.ok) {
                throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
            }
            return response.json();
        })
        .then(data => {
            // 驗證資料格式
            if (!data || typeof data !== 'object') {
                throw new Error("回應資料格式不正確");
            }
            return data;
        })
        .catch(error => {
            // 統一的錯誤處理
            console.error("資料載入失敗：", error);
            
            // 可以根據錯誤類型做不同處理
            if (error.message.includes("HTTP Error")) {
                // 伺服器錯誤
                showServerErrorMessage();
            } else if (error.message.includes("格式不正確")) {
                // 資料格式錯誤
                showDataFormatErrorMessage();
            } else {
                // 網路或其他錯誤
                showGenericErrorMessage();
            }
            
            // 重新拋出錯誤讓上層處理
            throw error;
        });
}
```

## 練習範例

```html
<script type="text/javascript">
	
	// 觸發 Promise
	function LaunchPromise() {
		
		console.log('LaunchPromise() 開始');
		var myPromise = createPromise();
		
		myPromise
			.then((result) => {
				console.log('成功：', result);
				
				// 這裡返回的字串會被自動包裝成已解決（resolved）的 Promise，然後傳遞給下一個 .then()
				// 相當於 return Promise.resolve('處理後的資料-XXX');
				// 如果沒有 return，就會自動變成 return Promise.resolve(undefined);
				// 每個 .then() 一定都會返回一個新的 Promise
				return '處理後的資料-XXX';	
			})
			.then((newResult)=> {
				console.log('進一步處理：', newResult);
			})
			.catch((error) => {
				console.log('錯誤：', error);
			})
			.finally(() => {
				console.log('清理工作完成');
				// 清除(停止) progressInterval
				clearInterval(progressInterval);
			});
		
		// 秒數計數，是異步作業
		var processCount = 0;
		var progressInterval = setInterval(() => {
			processCount++;
			console.log(`Promise 處理中 (${processCount}秒) ...`);
		}, 1000);		
	}
	
	// 建立 Promise
	function createPromise() {
		return new Promise((resolve, reject) => {
			var success;
			setTimeout(function () {
				success = true;
				if(success) {
					resolve('資料處理完成!!');					
				} else {
					resolve('資料處理失敗!!');
				}				
			},3000);
		});
	}
</script>

<input type="button" value="啟動 Promise" onclick="LaunchPromise()" style="margin: 10px;">
```

![[Pasted image 20250705233900.png]]

```html
<script type="text/javascript">
	
	function loginUser(credentials) {
		return new Promise((resolve, reject) => {
			// 模擬登入 API 呼叫
			setTimeout(() => {
				if (credentials.userName && credentials.passWord) {
					resolve({ token: "abc123", userId: 456 });
				} else {
					reject(new Error("登入資訊不完整"));
				}
			}, 2000);
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
			}, 2000);
		});
	}

	function LaunchChaining() {	
				
		// 秒數計數, 是異步作業
		var processCount = 0;
		var progressInterval = setInterval(() => {
			processCount++;
			console.log(`Promise 處理中 (${processCount}秒) ...`);
		}, 1000);
		
		console.log('LaunchPromise() 開始', `(${processCount}秒)`);		
		
		// 鏈接多個 Promise
		loginUser({ userName: "user", passWord: "pass" })
			.then((loginResult) => {
				console.log("登入成功，Token:", loginResult.token , `(${processCount}秒)`);
				// 回傳下一個 Promise 給下一個 .then()				
				return fetchUserProfile(loginResult.token, loginResult.userId);
			})
			.then((userProfile) => {
				console.log("使用者資料載入完成：", userProfile, `(${processCount}秒)`);				
				// updateUserInterface(userProfile);
				clearInterval(progressInterval);
			})
			.catch((error) => {
				console.error("操作過程發生錯誤：", error);				
			});
	}	

</script>

<input type="button" value="Promise 的鏈接" onclick="LaunchChaining()" style="margin: 10px;">
```

![[Pasted image 20250705234123.png]]

## 與 C# 的對比思考

作為 C# 開發者，你可能會發現 JavaScript Promise 與 C# 的 `Task` 和 `async/await` 有相似之處。JavaScript 的 Promise 就像 C# 的 Task，都代表一個異步操作的未來結果。在下一個階段的學習中，我們會探討 JavaScript 的 `async/await` 語法，它會讓你感覺更像在寫 C# 異步程式碼。
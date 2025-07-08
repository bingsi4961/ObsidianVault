---
date: 2025-07-07 10:19
aliases: 
tags:
  - HTTP
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

```javascript
fetch('https://jsonplaceholder.typicode.com/posts')
	.then(function(response) {
		
		// response 的型別： object
		console.log('response 的型別：', typeof response);
		
		// response 是否為 Response 實例: true
		console.log('response 是否為 Response 實例:', response instanceof Response);
		
		// response 是否為 Object 實例: true
		console.log('response 是否為 Object 實例:', response instanceof Object);
		
		// response 是否成功： true
		// 表示 HTTP 狀態碼是否在 200-299 範圍內
		console.log('response 是否成功：', response.ok);
		
		// response 狀態碼： 200
		// HTTP 狀態碼（如 200、404、500）
		console.log('response 狀態碼：', response.status);
		
		// response.statusText：
		// HTTP 狀態訊息（如 "OK"、"Not Found"）
		console.log('response.statusText：', response.statusText);
		
		//
		console.log('response：', response);
		
		// response.body
		console.log('response.body：', response.body);

		// 取得回應內容：從 Response 物件中讀取 HTTP 回應的 body 內容
	    // 解析 JSON：將回應內容（通常是 JSON 字串）解析成 JavaScript 物件
	    // 回傳 Promise：這是一個非同步操作，所以回傳 Promise
		var jsResponse = response.json();
		
		// response.json() 的型別： object
		console.log('response.json() 的型別：', typeof jsResponse);
		
		// response.json() 是否為 Promise 實例： true
		console.log('response.json() 是否為 Promise 實例：', jsResponse instanceof Promise);

		//
		console.log(jsResponse);
	});
```
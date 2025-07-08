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
		console.log('response 是否成功：', response.ok);
		
		// response 狀態碼： 200
		console.log('response 狀態碼：', response.status);
		
		// response.statusText：
		console.log('response.statusText：', response.statusText);
		
		//
		console.log('response：', response);
		
		// response.body
		console.log('response.body：', response.body);
				
		var jsResponse = response.json();
		
		// response.json() 的型別： object
		console.log('response.json() 的型別：', typeof jsResponse);
		
		// response.json() 是否為 Promise 實例： true
		console.log('response.json() 是否為 Promise 實例：', jsResponse instanceof Promise);

		//
		console.log(jsResponse);
	});
```
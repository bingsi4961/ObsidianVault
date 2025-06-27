---
date: 2025-06-27 10:02
aliases: 
tags:
  - CSS
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

```css
<style>

body {
	font-family: 'Microsoft JhengHei', sans-serif;
	margin: 20px;
	background-color: #f5f5f5;
}

.demo-box {
	background: white;
	padding: 20px;
	margin: 15px 0;
	border-radius: 8px;
	box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.code {
	background-color: #2d3748;
	color: #e2e8f0;
	padding: 10px;
	border-radius: 4px;
	font-family: monospace;
	margin: 10px 0;
}

.note {
	background-color: #fff3cd;
	border: 1px solid #ffeaa7;
	padding: 10px;
	border-radius: 4px;
	margin: 10px 0;
	font-size: 14px;
}

.block-item {
	background-color: #2196F3;
	padding: 5px;
	margin-bottom: 5px;
}

.inline-item {	
	background-color: #FF9800;
	padding: 5px;	
}

.inline-block-item {	
	background-color: #9C27B0;
	padding: 5px;
	margin-bottom: 5px;
}

.flex-container {	
	background-color: #f0f0f0;
	padding: 10px;
	border: 2px dashed #ccc;
}

.flex-item {
	background-color: #E91E63;
	color: white;
	padding: 15px;
	margin: 5px;
	text-align: center;
}

</style>
```

---
![[Pasted image 20250627103242.png]]

```html
<div class="demo-box">
	<h3>display: block (區塊元素)</h3>
	<div class="code">display: block;</div>
	<label class="block-item" style="display: block;">我是區塊元素 - 獨佔整行</label>
	<label class="block-item" style="display: block;">我也是區塊元素 - 也獨佔整行</label>
</div>
```


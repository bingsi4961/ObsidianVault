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


![[Pasted image 20250627103330.png]]

```html
<div class="demo-box">
	<h3>display: inline (行內元素)</h3>
	<div class="code">display: inline;</div>
	<div class="note">
		<div>依內容決定 div 寬度</div>
		<div>容器較小時，直接 <font color="red"><b>文字截斷換行</b></font>，div不會換行</div>
		<div>width、heigh、min-width、margin-top、margin-bottom 都無效</div>		
	</div>
	<div class="inline-item" style="display: inline;">行內元素，只佔據內容所需的寬度1</div>
	<div class="inline-item" style="display: inline;">行內元素，只佔據內容所需的寬度2</div>
	<div class="inline-item" style="display: inline;">行內元素，只佔據內容所需的寬度3</div>
</div>
```


![[Pasted image 20250627103418.png]]

```html
<div class="demo-box">
	<h3>display: inline-block (行內區塊)</h3>
	<div class="code">display: inline-block;</div>
	<div class="note">
		<div>依內容決定 div 寬度</div>
		<div>容器較小時，div 元素會換行</div>
	</div>
	<div class="inline-block-item" style="display: inline-block;"> 行內區塊元素，結合兩者特性1 </div>
	<div class="inline-block-item" style="display: inline-block;"> 行內區塊元素，結合兩者特性2 </div>
	<div class="inline-block-item" style="display: inline-block;"> 行內區塊元素，結合兩者特性3 </div>
</div>
```

📑 [[display 屬性 block、inline、inline-block]]


![[Pasted image 20250627103536.png]]
```html
<div class="demo-box">
	<h3>display: flex (彈性佈局)</h3>
	<div class="code">display: flex;</div>
	<div class="note">
		<div>注意 display: flex 是設定在容器。</div>
		<div>★ 子元素會水平排列。容器縮小時，子元素會自動調整寬度，不會換行。</div>
	</div>	
	
	<div class="flex-container" style="display: flex;">
		<div class="flex-item">彈性盒子佈局1彈性盒子佈局1彈性盒子佈局1</div>
		<div class="flex-item">彈性盒子佈局2彈性盒子佈局2彈性盒子佈局2</div>
		<div class="flex-item">彈性盒子佈局3彈性盒子佈局3彈性盒子佈局3</div>
	</div>
</div>
```


![[Pasted image 20250627103614.png]]

```html
<div class="demo-box">
	<h3>display: flex (彈性佈局) (設定 min-width)</h3>
	<div class="code">
		<div>display: flex;</div>
		<div>min-width: 200px;</div>
	</div>
	<div class="note">flex-item div 若有設定 min-width，則元素不會自動調整寬度，元素也不會換行</div>
	
	<div class="flex-container" style="display: flex;">
		<div class="flex-item" style="min-width: 200px;">彈性盒子佈局1</div>
		<div class="flex-item" style="min-width: 200px;">彈性盒子佈局2</div>
		<div class="flex-item" style="min-width: 200px;">彈性盒子佈局3</div>
	</div>
</div>
```
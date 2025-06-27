---
date: 2025-06-27 13:12
aliases: 
tags:
  - CSS
  - Flexbox
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

```html
<style>

	body {
		font-family: 'Microsoft JhengHei', sans-serif;
		margin: 20px;
		background-color: #f5f5f5;
	}
	
	.demo-box-3 {
		background: white;
		padding: 20px;
		margin: 15px 0;
		border-radius: 8px;
		box-shadow: 0 2px 8px rgba(0,0,0,0.1);
	}
	
	.flex-container-3 {
		background-color: #e8f4f8;
		padding: 15px;
		border: 2px solid #007acc;
		height: 120px;
		margin: 10px 0;
	}
	
	.code-3 {
		background-color: #2d3748;
		color: #e2e8f0;
		padding: 10px;
		border-radius: 4px;
		font-family: monospace;
		margin: 10px 0;
	}
	
	.note-3 {
		background-color: #fff3cd;
		border: 1px solid #ffeaa7;
		padding: 10px;
		border-radius: 4px;
		margin: 10px 0;
		font-size: 14px;
	}
	
	.flex-item-3 {
		background-color: #ff6b6b;
		color: white;
		padding: 15px;
		margin: 5px;
		text-align: center;
		border-radius: 4px;
	}

</style>
```

### flex-direction (排列方向)

![[Pasted image 20250627132058.png]]

```html
<h4>row (水平排列，預設)</h4>
<div class="code-3">
	<div>display: flex;</div>
	<div>flex-direction: row;</div>
</div>
<div class="note-3">align-items 預設值 stretch</div>
<div class="flex-container-3" style="display: flex; flex-direction: row;">
	<div class="flex-item-3">1</div>
	<div class="flex-item-3">2</div>
	<div class="flex-item-3">3</div>
</div>
```

![[Pasted image 20250627132135.png]]

```html
<h4>column (垂直排列)</h4>
<div class="code-3">
	<div>display: flex;</div>
	<div>flex-direction: column;</div>
	<div>height: 225px;</div>
</div>
<div class="note-3">因 flex-direction: column，並且 align-items 預設值 stretch，故滿版到最右邊</div>
<div class="flex-container-3" style="display: flex; flex-direction: column; height: 225px;">
	<div class="flex-item-3">1</div>
	<div class="flex-item-3">2</div>
	<div class="flex-item-3">3</div>
</div>
```

### 📑[[justify-content、align-items 容易混淆的觀念]]
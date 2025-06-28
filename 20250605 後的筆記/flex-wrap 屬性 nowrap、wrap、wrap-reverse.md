---
date: 2025-06-26 10:15
aliases: 
tags:
  - CSS
  - CSS_Flexbox
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
	
	.demo-box-2 {
		background: white;
		padding: 20px;
		margin: 15px 0;
		border-radius: 8px;
		box-shadow: 0 2px 8px rgba(0,0,0,0.1);
	}
	
	.code-2 {
		background-color: #2d3748;
		color: #e2e8f0;
		padding: 10px;
		border-radius: 4px;
		font-family: monospace;
		margin: 10px 0;
	}
	
	.note-2 {
		background-color: #fff3cd;
		border: 1px solid #ffeaa7;
		padding: 10px;
		border-radius: 4px;
		margin: 10px 0;
		font-size: 14px;
	}
	
	.flex-container-2 {	
		background-color: #e8f4f8;
		padding: 15px;
		border: 2px solid #007acc;
		margin: 10px 0;
		/*width: 400px;  固定寬度以便觀察換行效果 */
	}
	
	.flex-item-2 {
		background-color: #ff6b6b;
		color: white;
		padding: 15px;
		margin: 5px;
		text-align: center;
		border-radius: 4px;
		min-width: 100px; /* 設定最小寬度讓換行效果更明顯 */
	}

</style>
```

---

> [!NOTE] 記憶
> flex-wrap: wrap;		允許換行，項目會移到下一行
> flex-wrap: nowrap;		（預設）：不換行，項目會壓縮以適應容器
> flex-wrap: wrap-reverse	反向換行，新行出現在上方


![[Pasted image 20250627111756.png]]

```html
<div class="demo-box-2">
	<h3>flex-wrap: nowrap (預設值)</h3>
	<div class="code-2">
		<div>display: flex;</div>
		<div>flex-wrap: nowrap;</div>
	</div>
	<div class="note-2">
		<div>項目不會換行，會壓縮以適應容器寬度</div>
		<div>跟僅有設定 display: flex 是一樣的效果</div>
	</div>
	
	<div class="flex-container-2" style="display: flex; flex-wrap: nowrap;">
		<div class="flex-item-2">項目 1項目 1項目 1</div>
		<div class="flex-item-2">項目 2項目 2項目 2</div>
		<div class="flex-item-2">項目 3項目 3項目 3</div>
		<div class="flex-item-2">項目 4項目 4項目 4</div>
		<div class="flex-item-2">項目 5項目 5項目 5</div>
	</div>
</div>
```


![[Pasted image 20250627112205.png]]

```html
<div class="demo-box-2">
	<h3>flex-wrap: wrap</h3>
	<div class="code-2">
		<div>display: flex;</div>
		<div>flex-wrap: wrap;</div>
	</div>
	<div class="note-2">當容器寬度不足時，項目會換到下一行</div>
	
	<div class="flex-container-2" style="display: flex; flex-wrap: wrap;">
		<div class="flex-item-2">項目 1</div>
		<div class="flex-item-2">項目 2</div>
		<div class="flex-item-2">項目 3</div>
		<div class="flex-item-2">項目 4</div>
		<div class="flex-item-2">項目 5</div>
	</div>
</div>
```


![[Pasted image 20250627112306.png]]

```html
<div class="demo-box-2">
	<h3>flex-wrap: wrap-reverse</h3>
	<div class="code-2">
		<div>display: flex;</div>
		<div>flex-wrap: wrap-reverse;</div>
	</div>
	<div class="note-2">反向換行，新的行會出現在上方</div>
	
	<div class="flex-container-2" style="display: flex; flex-wrap: wrap-reverse;">
		<div class="flex-item-2">項目 1</div>
		<div class="flex-item-2">項目 2</div>
		<div class="flex-item-2">項目 3</div>
		<div class="flex-item-2">項目 4</div>
		<div class="flex-item-2">項目 5</div>
	</div>
</div>
```
---
date: 2025-06-27 11:25
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

## 預設行為（flex-wrap: nowrap）	
### 當容器寬度縮小時：

- 子元素會自動壓縮寬度，嘗試保持在同一行
- 子元素內的文字會自動換行，以配合縮小的寬度
- 當文字無法再換行壓縮時，子元素會溢出容器邊界
- 子元素本身不會換行到下一列

## 允許子元素換行（flex-wrap: wrap）
### 當不希望子元素被強制壓縮寬度時：

- 設定 flex-wrap: wrap
- 子元素會保持原本寬度，直接換行到下一列
- 避免子元素內容被過度壓縮或溢出容器


> [!NOTE] 記憶
> justify-content：主軸對齊方式（不一定是水平對齊）
> align-items：交叉軸對齊方式（不一定是垂直對齊）


> [!NOTE] justify-content 主軸屬性值
> justify-content: flex-start;	(預設) 置左/置頂
> justify-content: center;		置中
> justify-content: flex-end 		置右/置底
> justify-content: space-between;	兩端對齊

> [!NOTE] align-items 交叉軸屬性值
> align-items: flex-start	置左/置頂
> align-items: center		置中
> align-items: flex-end	置右/置底
> align-items: stretch	(預設) 拉伸填滿


### justify-content (主軸對齊)
- #### flex-direction 預設值 row
- #### align-items 預設值 stretch

![[Pasted image 20250627115543.png]]

```html
<h4>flex-start (預設)</h4>
<div class="code-3">
	<div>display: flex;</div>
	<div>justify-content: flex-start;</div>
</div>
<div class="note-3">
	<div>flex-direction 預設值是 row</div>
	<div>align-items 預設值是 stretch</div>
</div>	
<div class="flex-container-3" style="display: flex; justify-content: flex-start;">
	<div class="flex-item-3">1</div>
	<div class="flex-item-3">2</div>
	<div class="flex-item-3">3</div>
</div>
```

![[Pasted image 20250627115617.png]]

```html
<h4>center (置中)</h4>
<div class="code-3">
	<div>display: flex;</div>
	<div>justify-content: center;</div>
</div>
<div class="flex-container-3" style="display: flex; justify-content: center;">
	<div class="flex-item-3">1</div>
	<div class="flex-item-3">2</div>
	<div class="flex-item-3">3</div>
</div>
```

![[Pasted image 20250627115654.png]]

```html
<h4>center (置右)</h4>
<div class="code-3">
	<div>display: flex;</div>
	<div>justify-content: flex-end;</div>
</div>
<div class="flex-container-3" style="display: flex; justify-content: flex-end;">
	<div class="flex-item-3">1</div>
	<div class="flex-item-3">2</div>
	<div class="flex-item-3">3</div>
</div>
```

![[Pasted image 20250627121004.png]]

```html
<h4>space-between (兩端對齊)</h4>
	<div class="code-3">
		<div>display: flex;</div>
		<div>justify-content: space-between;</div>
	</div>
	<div class="flex-container-3" style="display: flex; justify-content: space-between;">
		<div class="flex-item-3">1</div>
		<div class="flex-item-3">2</div>
		<div class="flex-item-3">3</div>
	</div>
```

![[Pasted image 20250627121039.png]]
```html
<h4>space-between (兩端對齊 && 空間平均分配)</h4>
<div class="code-3">
	<div>display: flex;</div>
	<div>justify-content: space-between;</div>
	<div>flex:1</div>
</div>
<div class="flex-container-3" style="display: flex; justify-content: space-between;">
	<div class="flex-item-3" style="flex:1">1</div>
	<div class="flex-item-3" style="flex:1">2</div>
	<div class="flex-item-3" style="flex:1">3</div>
</div>
```
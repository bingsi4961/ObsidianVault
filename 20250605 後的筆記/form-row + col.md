---
date: 2025-06-25 14:13
aliases: 
tags:
  - BootStrap
  - BootStrap_4_3_1
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## 程式碼結構分析

```html
<form>
  <div class="form-row">
    <div class="col">
      <input type="text" class="form-control" placeholder="名">
    </div>
    <div class="col">
      <input type="text" class="form-control" placeholder="姓">
    </div>
  </div>
</form>
```

## 各個 CSS 類別說明

### `form-row`

- Bootstrap 4 專門為 ==表單設計== 的網格系統
- 類似 `row` 類別，但針對表單欄位優化
- ==提供更緊密的欄位間距（gutters 較小）==
- 確保表單欄位在同一列水平對齊

### `col`

- Bootstrap 的網格欄位類別
- 在 `form-row` 中會自動平分可用空間
- 這裡兩個 `col` 會各佔 50% 寬度
- 響應式設計，會根據螢幕大小自動調整
- 📑 [[col-6 與 col-md-6 的差異]]

### `form-control`

- Bootstrap 的標準表單控制項樣式
- 提供統一的外觀：邊框、內距、字體大小等
- 包含焦點狀態、禁用狀態等視覺效果
- ==自動適應父容器寬度（width: 100%）==

## 擴展應用

如果你想要不同的欄位寬度比例，可以使用：

```html
<div class="col-8">  <!-- 佔 8/12 = 2/3 寬度 -->
<div class="col-4">  <!-- 佔 4/12 = 1/3 寬度 -->
```

---

```html
<form>
	<div class="form-row">
		<div class="col-3">
			<input type="text" class="form-control" placeholder="名">
		</div>
		<div class="col-3">
			<input type="text" class="form-control" placeholder="姓">
		</div>
		<div class="col-6">
			<input type="text" class="form-control" placeholder="座右銘">
		</div>
	</div>
	<div class="form-row">
		<div class="col-4">
			<input type="text" class="form-control" placeholder="座右銘">
		</div>
		<div class="col-4">
			<input type="text" class="form-control" placeholder="名">
		</div>
		<div class="col-4">
			<input type="text" class="form-control" placeholder="姓">
		</div>		
	</div>
</form>
```

![[Pasted image 20250625143406.png]]

加上 mt-3 讓兩行 Row 產生一點間距

```html
<form>
	<div class="form-row">
		<div class="col-3">
			<input type="text" class="form-control" placeholder="名">
		</div>
		<div class="col-3">
			<input type="text" class="form-control" placeholder="姓">
		</div>
		<div class="col-6">
			<input type="text" class="form-control" placeholder="座右銘">
		</div>
	</div>
	<div class="form-row mt-3">
		<div class="col-4">
			<input type="text" class="form-control" placeholder="座右銘">
		</div>
		<div class="col-4">
			<input type="text" class="form-control" placeholder="名">
		</div>
		<div class="col-4">
			<input type="text" class="form-control" placeholder="姓">
		</div>		
	</div>
</form>
```

![[Pasted image 20250625143612.png]]
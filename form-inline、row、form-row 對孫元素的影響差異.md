---
date: 2025-06-27 22:44
aliases:
  - 別名測試1
  - 別名測試2
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

## 核心概念

### 影響範圍的差異

- **form-inline**：影響所有子元素和孫級元素（全部水平排列）
- **row**：只影響直接子元素（孫元素垂直排列保持不變）
- **form-row**：只影響直接子元素（孫元素垂直排列保持不變）

## 各類別詳細分析

### 1. form-inline

```css
.form-inline {
    display: flex;
    flex-flow: row wrap;
	/*
		flex-direction: row;
		flex-wrap: wrap;    
	*/
    align-items: center;
}

/* 關鍵：Bootstrap 額外定義子元素行為 */
.form-inline .form-control, /* .form-line 下層的 .form-control */
.form-inline .form-group,
.form-inline .input-group,
.form-inline .form-check {
    display: flex; /* 或 inline-block */
    align-items: center;
    margin-bottom: 0;
}

.form-inline label {
    margin-bottom: 0;
    margin-right: 0.5rem;
}
```

**特點：**

- 所有表單元素強制水平排列
- Bootstrap 定義了豐富的子元素樣式規則
- 適合簡單的單行表單

### 2. row

```css
.row {
    display: flex;
    flex-wrap: wrap;
    margin-right: -15px;  /* 標準 gutter */
    margin-left: -15px;
}
```

**特點：**

- 只影響直接子元素（通常是 col-md-* 類別）
- 15px 的標準間距
- 子元素內部保持垂直排列
- 通用的網格佈局容器

### 3. form-row

```css
.form-row {
    display: flex;
    flex-wrap: wrap;
    margin-right: -5px;   /* 較小的 gutter */
    margin-left: -5px;
}
```

**特點：**

- 只影響直接子元素（通常是 col-* 類別）
- 5px 的較小間距，專為表單設計
- 子元素內部保持垂直排列
- 專門用於表單的網格佈局

## 實際範例對比

```javascript
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">	
<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
```

### form-inline 範例

```html
<form class="form-inline">
	<div class="form-group mr-3">
	    <label for="email" class="mr-2">Email:</label>
	    <input type="email" class="form-control" id="email">
	</div>
	<div class="form-group mr-3">
	    <label for="password" class="mr-2">Password:</label>
	    <input type="password" class="form-control" id="password">
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
```

![[Pasted image 20250628091215.png]]
### row / form-row 範例

```html
<form class="form-row">
	<div class="form-group mr-3">
	    <label for="email" class="mr-2">Email:</label>
	    <input type="email" class="form-control" id="email">
	</div>
	<div class="form-group mr-3">
	    <label for="password" class="mr-2">Password:</label>
	    <input type="password" class="form-control" id="password">
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
```

![[Pasted image 20250628095639.png]]

## 關鍵差異總結

|類別|容器類型|影響範圍|間距|用途|
|---|---|---|---|---|
|form-inline|Flexbox|所有層級元素|-|單行水平表單|
|row|Flexbox|僅直接子元素|15px|通用網格佈局|
|form-row|Flexbox|僅直接子元素|5px|表單專用網格佈局|

## 除錯技巧

### 使用 F12 開發者工具查看樣式

1. **檢查元素**：右鍵 → 檢查元素
2. **查看 Computed 樣式**：切換到 "Computed" 分頁
3. **查看 Styles 面板**：在 "Styles" 分頁找到相關 CSS 規則

### 常見的 form-inline 子元素規則

```css
.form-inline .form-control {
    display: inline-block;
    width: auto;
    vertical-align: middle;
}

.form-inline .form-group {
    display: inline-block;
    margin-bottom: 0;
    vertical-align: middle;
}
```

## 使用建議

- **form-inline**：適用於搜尋框、登入表單等簡單的水平排列需求
- **row**：適用於一般的網格佈局，不限於表單
- **form-row**：適用於複雜表單的多欄位排列，間距較緊密適合表單元素
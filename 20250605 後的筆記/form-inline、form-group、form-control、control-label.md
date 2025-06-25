---
date: 2025-06-11 11:37
aliases: 
tags:
  - BootStrap
  - BootStrap_3
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
## form-inline

- 應用在 `<form>` 元素上的 CSS 類別，用來讓表單控制項，以水平排列的方式在同一行顯示。
- 只在 768px 以上 的螢幕寬度才會生效。這是 Bootstrap 的響應式設計，在小螢幕（手機）上會自動變回垂直排列。
- 確保父容器有足夠寬度容納所有表單元素
- 包含了 d-flex 的功能
- 📑 [[form-inline 應該包在 form-group 外面]]
## form-group

- 用於包裝表單控制項和標籤，提供適當的==間距==和結構。這是 Bootstrap 表單的基本容器單位。
- 會讓裡面的表單元件以==垂直方向排列==（換行顯示）
- 📑 [[不是說 form-group 會讓裡面的表單元件保持適當距離嗎？]]
## form-control

- 應用於表單輸入元素（input、textarea、select），提供統一的樣式設計，包括邊框、內距、焦點效果等。
- 會套用 [[display block、display inline-block|display: block]]，讓元素變成區塊級元素，佔據整行寬度，強制換行
- ==預設會讓元件 width: 100%==
## control-label

- 用於表單標籤的樣式類別，確保標籤與表單控制項有適當的對齊和間距。

***有設定 class="form-inline"*** 
```html
<form class="form-inline">
	<div class="form-group">
		<label for="email" class="control-label" >Email:</label>
		<input type="email" class="form-control" id="email">
	</div>
	<div class="form-group">
		<label for="pwd" class="control-label" >Password:</label>
		<input type="password" class="form-control" id="pwd">
	</div>
	<div class="form-group">
		<label class="control-label">其他輸入：</label>
		<input type="text" class="form-control" placeholder="文字輸入框">
		<textarea class="form-control" rows="3" placeholder="多行文字區域"></textarea>
		<select class="form-control">
			<option>選項 1</option>
			<option>選項 2</option>
		</select>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
```

![[Pasted image 20250611114610.png]]

***沒有設定 class="form-inline"***
```html
<form>
	<div class="form-group">
		<label for="email" class="control-label" >Email:</label>
		<input type="email" class="form-control" id="email">
	</div>
	<div class="form-group">
		<label for="pwd" class="control-label" >Password:</label>
		<input type="password" class="form-control" id="pwd">
	</div>
	<div class="form-group">
		<label class="control-label">其他輸入：</label>
		<input type="text" class="form-control" placeholder="文字輸入框">
		<textarea class="form-control" rows="3" placeholder="多行文字區域"></textarea>
		<select class="form-control">
			<option>選項 1</option>
			<option>選項 2</option>
		</select>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>

```

![[Pasted image 20250611115110.png]]

## 完整範例

結合以上所有類別的實際應用：

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css">
</head>
<body>
  <div class="container">
    <!-- 垂直表單 -->
    <form>
      <div class="form-group">
        <label class="control-label" for="name">姓名</label>
        <input type="text" class="form-control" id="name" placeholder="請輸入姓名">
      </div>
      <div class="form-group">
        <label class="control-label" for="email">電子郵件</label>
        <input type="email" class="form-control" id="email" placeholder="請輸入電子郵件">
      </div>
    </form>

    <!-- 水平表單 -->
    <form class="form-inline">
      <div class="form-group">
        <label class="control-label" for="search">搜尋：</label>
        <input type="text" class="form-control" id="search" placeholder="搜尋關鍵字">
      </div>
      <button type="submit" class="btn btn-primary">搜尋</button>
    </form>
  </div>
</body>
</html>
```

這些類別通常搭配使用，form-group 作為容器，control-label 設定標籤樣式，form-control 美化輸入元件，而 form-inline 則控制整體佈局方向。
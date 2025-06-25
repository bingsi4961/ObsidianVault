---
date: 2025-06-18 10:37
aliases: []
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

Bootstrap 的 input-group 功能讓你可以在 ==輸入欄位== 前後加上文字、按鈕或其他元素，創造出更美觀且實用的表單介面。

### 基本結構

```html
<div class="input-group">
  <div class="input-group-prepend">
    <span class="input-group-text">@</span>
  </div>
  <input type="text" class="form-control" placeholder="使用者名稱">
</div>
```

### 主要類別說明

**`.input-group`**

- 包裹整個輸入群組的容器
- 使用 flexbox 佈局，讓內部元素水平排列
- 包含了 display: flex 的樣式設定

**`.input-group-text`**

- 用於顯示純文字內容的裝飾元素
- 通常放在 `.input-group-prepend` 或 `.input-group-append` 內
- 會自動調整樣式以配合輸入欄位

**`.input-group-append`**

- 將內容附加到輸入欄位的右側（後方）
- 在 Bootstrap 5 中已被移除，改用其他方式

```html
<!-- Bootstrap CSS -->
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
	
<div class="p-2">
	<div class="form-group">
		<div class="input-group">
			<div class="input-group-prepend">
				<span class="input-group-text">@</span>
			</div>
			<input type="text" class="form-control" placeholder="使用者名稱">
			<div class="input-group-append">
				<span class="input-group-text">芭樂888塊</span>
			</div>
		</div>
	</div>
	<div class="form-group">
		<div class="input-group">
			<input type="text" class="form-control" placeholder="請輸入搜尋關鍵字">
			<div class="input-group-append">
				<input type="button" class="btn btn-primary" value="搜尋" />
			</div>
		</div>	
	</div>
	<div class="form-group">
		<div class="input-group">
			<div class="input-group-prepend">
				<span class="input-group-text">💰</span>
				<span class="input-group-text">NT$</span>
			</div>
			<!--
			<div class="input-group-prepend">
				<span class="input-group-text">NT$</span>
			</div>
			-->
			<input type="number" class="form-control" placeholder="0">
			<div class="input-group-append">
				<span class="input-group-text">.00</span>
			</div>
		</div>
	</div>
	
	<script type="text/JavaScript">
	
		$(document).ready(function() {
			$('.dropdown-item').click(function(e) {
				e.preventDefault(); // 防止頁面跳轉
				
				var selectedText = $(this).text();
				var selectedValue = $(this).data('value');
				
				// 更新按鈕文字
				$('#categoryButton').text(selectedText);
				
				// 可以儲存選中的值到隱藏欄位或變數
				// 例如：執行搜尋、篩選等操作
				console.log('選擇的分類：', selectedValue);
			});
		});
	
	</script>
	
	<div class="form-group">
		<div class="input-group">
			<div class="input-group-prepend">
				<button class="btn btn-outline-secondary dropdown-toggle" type="button" data-toggle="dropdown" id="categoryButton">
					選擇分類
				</button>
				<div class="dropdown-menu">
					<a class="dropdown-item" href="#" data-value="product">產品</a>
					<a class="dropdown-item" href="#" data-value="service">服務</a>
				</div>
			</div>
			<input type="text" class="form-control" placeholder="輸入搜尋內容" id="searchInput">
		</div>
	</div>
</div>

<!-- jQuery -->
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<!-- Popper.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js"></script>
<!-- Bootstrap JavaScript -->
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js"></script>
```

![[Pasted image 20250618105402.png]]

```html
<div class="form-group">
	<div class="input-group">
		<!--<div class="input-group-prepend">-->
			<span class="input-group-text">💰</span>
			<span class="input-group-text">NT$</span>
		<!--</div>-->		
		<input type="number" class="form-control" placeholder="0">
		<!--<div class="input-group-append">-->
			<span class="input-group-text">.00</span>
		<!--</div>-->
	</div>
</div>
```

若沒有使用 input-group-prepend、input-group-append 包起來的話，會有圓角

![[Pasted image 20250618105707.png]]

📑BootStrap 3 的用法 [[input-group、input-group-text、btn-group]]
📑BootStrap 5 的用法 [[input-group-addon]]
---
date: 2025-06-15 23:31
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
📑[[BootStrap 網格系統]]
📑[[form-inline、form-group、form-control、control-label]]
📑[[input-group、input-group-text、input-group-prepend、input-group-append]]
📑[[input-group、input-group-text、btn-group]]
📑[[Flexbox 佈局]]
📑[[設定 Padding]]
📑[[寬度設定 w-number]]

```html
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">	
<script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>

<div class="container mt-2">
	<div class="row">
		<div class="col-md-6">
			<div class="form-group"> <!-- form-group 會讓元件保持適當的間距，及以垂直方向排列 -->
				<label for="txtIdentity" class="control-label">身份證號碼：</label>
				<input type="text" id="txtIdentity" placeholder="身份證號碼" class="form-control">
				<input type="button" value="搜尋" class="btn btn-primary mt-1">
				<!-- button 不應該用 form-control -->
			</div>
		</div>
		<div class="col-md-6">
			<!-- 因 form-group 會讓元件以垂直方向排列，故使用 form-inline 讓元件再以水平方向排列 -->
			<div class="form-group form-inline">
				<label for="txtIdentity" class="control-label">身份證號碼：</label>
				<input type="text" id="txtIdentity" placeholder="身份證號碼" class="form-control">
				<input type="button" value="搜尋" class="btn btn-primary ml-1">
			</div>
		</div>
	</div>
	<div class="row">
		<div class="col-md-6">
			<div class="form-group form-inline"> <!-- form-line 有包含 d-flex 的功能 -->
				<div class="input-group flex-fill">
					<div class="input-group-prepend">
						<span class="input-group-text rounded-0">https：//</span>
					</div>
					<input type="text" id="txtURI" class="form-control" placeholder="網站名稱">
					<div class="input-group-append">
						<span class="input-group-text rounded-0">.com</span>
					</div>
				</div>
				<input type="button" value="搜尋" class="btn btn-primary ml-1">
				<!--
				<div class="input-group w-100">				
					<div class="input-group-prepend">
						<span class="input-group-text rounded-0">https：//</span>
					</div>
					<input type="text" id="txtURI" class="form-control" placeholder="網站名稱">
					<div class="input-group-append">
						<span class="input-group-text rounded-0">.com</span>
					</div>
					<input type="button" value="搜尋" class="btn btn-primary ml-1">
				</div>
				-->
			</div>
		</div>
		<div class="col-md-3 p-1 d-flex border">
			<!-- col-md-* 預設有左右 padding ，可透過 px-1 來設定 padding -->
			<div class="flex-fill border">項目 1</div>
			<div class="flex-fill border">項目 2</div>
			<div class="flex-fill border">項目 3</div>			
		</div>
		<div class="col-md-3">
			
		</div>
	</div>	
<div>
```

![[Pasted image 20250615233912.png]]
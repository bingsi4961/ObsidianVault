---
date: 2025-06-25 15:01
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

```html
<div class="card">
	<div class="card-header">
	    <h5>用戶資料</h5>
	</div>
	<div class="card-body">
	    <div class="form-row">
			<div class="col-md-6">
				<input type="text" class="form-control" placeholder="名">
			</div>
			<div class="col-md-6">
		        <input type="text" class="form-control" placeholder="姓">
			</div>
		</div>
	</div>
</div>
```

![[Pasted image 20250625163454.png]]

```html
<div class="card">
	<div class="card-header">
		<h5 class="mb-0">用戶資料</h5>
	</div>
	<div class="card-body">
		<!-- 姓名 -->
		<div class="form-row mb-3">
			<div class="col-6">
				<label for="firstName">名字 <span class="text-danger">*</span></label>
				<input type="text" class="form-control" id="firstName" placeholder="請輸入名字" required>
			</div>
			<div class="col-6">
				<label for="lastName">姓氏 <span class="text-danger">*</span></label>
				<input type="text" class="form-control" id="lastName" placeholder="請輸入姓氏" required>
			</div>
		</div>
		
		<!-- 性別與生日 -->
		<div class="form-row mb-3">
			<div class="col-4">
				<label for="gender">性別</label>
				<select class="form-control" id="gender">
					<option value="">請選擇</option>
					<option value="male">男性</option>
					<option value="female">女性</option>
					<option value="other">其他</option>
					<option value="prefer_not_to_say">不願透露</option>
				</select>
			</div>
			<div class="col-8">
				<label for="birthDate">生日</label>
				<input type="date" class="form-control" id="birthDate">
			</div>
		</div>		

		<!-- 詳細地址 -->
		<div class="form-group mb-3">
			<label for="address">詳細地址</label>
			<input type="text" class="form-control" id="address" placeholder="請輸入詳細地址">
		</div>		

		<!-- 備註 -->
		<div class="form-group mb-3">
			<label for="notes">備註</label>
			<textarea class="form-control" id="notes" rows="3" placeholder="其他需要說明的資訊"></textarea>
		</div>

		<!-- 同意條款 -->
		<div class="form-check mb-3">
			<input class="form-check-input" type="checkbox" id="agreeTerms" required>
			<label class="form-check-label" for="agreeTerms">
				我同意<a href="#" class="text-primary">服務條款</a>與<a href="#" class="text-primary">隱私權政策</a> 
				<span class="text-danger">*</span>
			</label>
		</div>

		<!-- 按鈕 -->
		<div class="form-row">
			<div class="col-6">
				<button type="button" class="btn btn-secondary btn-block">取消</button>
			</div>
			<div class="col-6">
				<button type="submit" class="btn btn-primary btn-block">儲存資料</button>
			</div>
		</div>
	</div>
</div>
```

![[Pasted image 20250625163624.png]]
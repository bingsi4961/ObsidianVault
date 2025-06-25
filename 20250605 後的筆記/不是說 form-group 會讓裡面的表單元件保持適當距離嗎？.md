---
date: 2025-06-25 12:01
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

`form-group` 在**一般情況下**確實會為內部元件提供適當的間距，但在 `form-inline` 的情況下行為會有所不同。

**一般表單（垂直排列）：**

```html
<form>
	<div class="form-group">
	    <label for="email">Email:</label>
	    <input type="email" class="form-control" id="email">
  </div>
  <div class="form-group">
		<label for="password">Password:</label>
		<input type="password" class="form-control" id="password">
  </div>
</form>
```

在這種情況下，`form-group` 會自動在每個群組之間添加 `margin-bottom`。

![[Pasted image 20250625115924.png]]

**但在 form-inline 中：**

```html
<form class="form-inline">
	<div class="form-group">
	    <label for="email">Email:</label>
	    <input type="email" class="form-control" id="email">
	</div>
	<div class="form-group">
	    <label for="password">Password:</label>
	    <input type="password" class="form-control" id="password">
	</div>
</form>
```

當套用 `form-inline` 時，Bootstrap 會：

1. 將 `form-group` 的 `margin-bottom` 重設為 0
2. 改為水平排列，但**不會自動添加水平間距**

![[Pasted image 20250625120023.png]]

這就是為什麼在 `form-inline` 中需要手動添加 `mr-2`、`mr-3` 等間距類別的原因。

所以您的理解是對的，但 `form-inline` 會覆蓋掉 `form-group` 原本的間距行為，這是 Bootstrap 設計上的考量，讓開發者可以更靈活地控制水平排列時的間距。
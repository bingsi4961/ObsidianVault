---
date: 2025-06-25 11:41
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

在 Bootstrap 中，`form-inline` 應該包在 `form-group` **外面**。
正確的結構是：

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

![[Pasted image 20250625120118.png]]

**原因說明：**

1. `form-inline` 是應用在 `<form>` 元素上的 CSS 類別，用來讓整個表單以水平排列的方式顯示
2. `form-group` 是用來包裝個別的表單控制項（label + input），提供適當的間距和佈局
3. 當 `form-inline` 套用在父層的 `<form>` 上時，它會影響內部所有 `form-group` 的排列方式

如果你把 `form-inline` 放在 `form-group` 裡面，會破壞 Bootstrap 預期的 DOM 結構，可能導致樣式無法正確套用。

📖 [[不是說 form-group 會讓裡面的表單元件保持適當距離嗎？]]

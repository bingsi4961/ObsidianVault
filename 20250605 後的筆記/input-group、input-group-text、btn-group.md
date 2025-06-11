---
date: 2025-06-11 09:17
aliases: 
tags:
  - BootStrap
  - BootStrap_5
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
## input-group（輸入群組）

`input-group` 用來將輸入欄位與按鈕或其他元素組合在一起，創造出視覺上連接的效果。

```html
<div class="input-group">
  <span class="input-group-text">https://</span>
  <input type="text" class="form-control" placeholder="網站名稱">
  <span class="input-group-text">.com</span>
</div>
```

![[Pasted image 20250611093407.png]]

```html
<div class="input-group">
  <input type="text" class="form-control" placeholder="搜尋...">
  <button class="btn btn-primary" type="button">搜尋</button>
</div>
```

![[Pasted image 20250611095348.png]]

📑BootStrap 3 的用法 [[input-group-addon]]

## btn-group（按鈕群組）

`btn-group` 用來將多個按鈕組合成一個群組，按鈕之間會緊密連接。

```html
<div class="btn-group" role="group">
  <button type="button" class="btn btn-primary">左</button>
  <button type="button" class="btn btn-primary">中</button>
  <button type="button" class="btn btn-primary">右</button>
</div>
```

![[Pasted image 20250611101011.png]]

```html
<div class="btn-group" >
  <button type="button" class="btn btn-outline-secondary rounded-0">左</button>
  <button type="button" class="btn btn-outline-secondary rounded-0">中</button>
  <button type="button" class="btn btn-outline-secondary rounded-0">右</button>
</div>
```

![[Pasted image 20250611101134.png]]
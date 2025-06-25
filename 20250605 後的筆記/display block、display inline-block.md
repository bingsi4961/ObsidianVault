---
title: display:block、display:inline-block
date: 2025-06-25 15:26
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

## display: block（區塊級元素）

**特性：**

- ==會佔據整行的寬度（即使內容不需要那麼寬）==
- 高度由內容決定
- 可以設定 width、height、margin、padding
- ==會自動換行（前後都會換行）==

**常見的 block 元素：**

```html
<div>、<p>、<h1>~<h6>、<ul>、<li>、<form>
```

**範例：**

```html
<div style="background: lightblue;">區塊1</div>
<div style="background: lightgreen;">區塊2</div>
```

結果：兩個 div 會垂直排列，各佔一整行

## display: inline-block（行內區塊元素）

**特性：**

- ==寬度由內容決定（不會佔據整行）==
- 可以設定 width、height、margin、padding
- ==不會自動換行，可以並排顯示==
- 結合了 inline 和 block 的優點

**範例：**

```html
<span style="display: inline-block; background: lightblue; width: 100px; height: 50px;">元素1</span>
<span style="display: inline-block; background: lightgreen; width: 100px; height: 50px;">元素2</span>
```

結果：兩個元素會水平並排

## 實際應用場景

在你的 C# 開發中，當你需要建立表單時：

```html
<!-- block: 每個輸入框佔一行 -->
<label>姓名</label>
<input type="text" class="form-control" style="display: block;">
<label>信箱</label>
<input type="email" class="form-control" style="display: block;">

<!-- inline-block: 可以並排 -->
<label>姓名</label>
<input type="text" style="display: inline-block; width: 200px;">
<label>年齡</label>
<input type="number" style="display: inline-block; width: 100px;">
```

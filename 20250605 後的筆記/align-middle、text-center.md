---
date: 2025-06-24 13:46
aliases: 
tags:
  - BootStrap_4_3_1
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## `text-center`（水平置中）

```css
.text-center {
    text-align: center !important;
}
```

- 讓內容在水平方向置中
- 適用於 inline 或 inline-block 元素（如文字、圖片、checkbox 等）

## `align-middle`（垂直置中）

```css
.align-middle {
    vertical-align: middle !important;
}
```

- 讓內容在垂直方向置中
- 適用於 table cell 或 inline/inline-block 元素

## 實際效果

```html
<!-- 只有水平置中 -->
<td class="text-center">
    <input type="checkbox">
</td>

<!-- 只有垂直置中 -->
<td class="align-middle">
    <input type="checkbox">
</td>

<!-- 水平 + 垂直置中 -->
<td class="text-center align-middle">
    <input type="checkbox">
</td>
```

## Bootstrap 其他相關類別

### 垂直對齊

```css
.align-baseline    { vertical-align: baseline !important; }
.align-top        { vertical-align: top !important; }
.align-middle     { vertical-align: middle !important; }
.align-bottom     { vertical-align: bottom !important; }
.align-text-bottom { vertical-align: text-bottom !important; }
.align-text-top   { vertical-align: text-top !important; }
```

### 水平對齊

```css
.text-left    { text-align: left !important; }
.text-right   { text-align: right !important; }
.text-center  { text-align: center !important; }
.text-justify { text-align: justify !important; }
```

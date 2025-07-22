---
date: 2025-06-15 22:46
aliases: 
tags:
  - BootStrap_4_3_1
---
 Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

`px-0` 是 Bootstrap 的 **padding 工具類別**，用來移除元素左右兩側的內距。

## 語法說明

- `p` = padding（內距）
- `x` = x軸方向（左右兩側）
- `0` = 0px（完全移除）

## 詳細說明

```html
<div class="px-0">
    這個元素的左右 padding 都是 0
</div>
```

等同於 CSS：

```css
.px-0 {
    padding-left: 0 !important;
    padding-right: 0 !important;
}
```

## Bootstrap Spacing 系統

Bootstrap 有完整的間距系統：

**方向代碼：**

- `t` = top（上）
- `b` = bottom（下）
- `l` = left（左）
- `r` = right（右）
- `x` = 左右兩側
- `y` = 上下兩側
- 無方向 = 四個方向

**數值：**

- `0` = 0px
- `1` = 0.25rem
- `2` = 0.5rem
- `3` = 1rem
- `4` = 1.5rem
- `5` = 3rem

## 常見用法範例

```html
<!-- 移除所有 padding -->
<div class="p-0">內容</div>

<!-- 只移除左右 padding -->
<div class="px-0">內容</div>

<!-- 只移除上下 padding -->
<div class="py-0">內容</div>

<!-- 移除左側 padding -->
<div class="pl-0">內容</div>
```

## 實際應用場景

在你的 .NET MVC 專案中，`px-0` 常用於：

```html
<!-- 讓內容貼近容器邊緣 -->
<div class="container-fluid px-0">
    <div class="row">
        <div class="col-12 px-0">
            <!-- 圖片或內容完全貼邊 -->
        </div>
    </div>
</div>

<!-- 移除 Bootstrap 欄位的預設間距 -->
<div class="col-md-6 px-0">
    <img src="image.jpg" class="w-100">
</div>
```

這樣可以讓元素完全貼近邊緣，常用於輪播圖、背景圖片或需要無縫拼接的布局。
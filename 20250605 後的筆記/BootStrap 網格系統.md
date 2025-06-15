---
date: 2025-06-11 10:53
aliases: 
tags:
  - BootStrap
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
## 基本概念

Bootstrap 網格系統基於 12 欄的彈性佈局，使用一系列的容器、列和欄來排版內容。它建立在 CSS Flexbox 之上，完全響應式。

## 核心組件

**容器 (Container)**

- `.container`: 固定寬度容器，在不同螢幕尺寸下有最大寬度限制
- `.container-fluid`: 全寬度容器，佔滿整個viewport寬度

**列 (Row)**

- `.row`: 包裝欄位的水平群組
- 每個 row 最多包含 12 個欄位單位

**欄 (Column)**

- `.col-*`: 定義欄位寬度和響應式行為
- `.col-*`: 預設有左右 padding（通常是 15px），這會影響內部元素的顯示，可使用 px-0 來消除。🔖[[設定 Padding]]

## 基本結構

```html
<div class="container">
  <div class="row">
    <div class="col-md-4">欄位 1</div>
    <div class="col-md-4">欄位 2</div>
    <div class="col-md-4">欄位 3</div>
  </div>
</div>
```

## 響應式斷點

Bootstrap 5 提供以下斷點：

- `xs` (< 576px): 超小螢幕
- `sm` (≥ 576px): 小螢幕
- `md` (≥ 768px): 中等螢幕
- `lg` (≥ 992px): 大螢幕
- `xl` (≥ 1200px): 超大螢幕
- `xxl` (≥ 1400px): 超超大螢幕

※ ==若螢幕的寬度小於設定的斷點，則 < div class="row" > 內的 div 元件以垂直呈現。==

## 欄位類別

**等寬欄位**

```html
<div class="row">
  <div class="col">1 of 2</div>
  <div class="col">2 of 2</div>
</div>
```

**指定寬度**

```html
<div class="row">
  <div class="col-6">佔 6 欄</div>
  <div class="col-6">佔 6 欄</div>
</div>
```

**響應式組合**

```html
<div class="row">
  <div class="col-12 col-md-8">手機全寬，桌面佔 8 欄</div>
  <div class="col-6 col-md-4">手機半寬，桌面佔 4 欄</div>
</div>
```

## 實用功能

**垂直對齊**

```html
<div class="row align-items-center">
  <div class="col">垂直置中</div>
</div>

<div class="row">
  <div class="col align-self-start">對齊頂部</div>
  <div class="col align-self-center">對齊中間</div>
  <div class="col align-self-end">對齊底部</div>
</div>
```

## 常見應用模式

**典型網站佈局**

```html
<div class="container">
    <!-- 標題列 -->
    <div class="row">
        <div class="col-12">
            <header>網站標題</header>
        </div>
    </div>
    
    <!-- 主要內容 -->
    <div class="row">
        <div class="col-md-8">
            <main>主要內容</main>
        </div>
        <div class="col-md-4">
            <aside>側邊欄</aside>
        </div>
    </div>
    
    <!-- 頁尾 -->
    <div class="row">
        <div class="col-12">
            <footer>版權資訊</footer>
        </div>
    </div>
</div>
```

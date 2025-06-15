---
date: 2025-06-15 23:12
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
## 基本概念

### d-flex

- **功能**：啟用 Flexbox 容器
- **等同於**：CSS 的 `display: flex`
- **重要觀念**：只有 `d-flex` 不會讓子元素自動平均分配寬度

```html
<!-- 基本用法 -->
<div class="d-flex border">
    <div class="flex-fill border">項目 1</div>
    <div class="flex-fill border">項目 2</div>
    <div class="flex-fill border">項目 3</div>
</div>
```

![[Pasted image 20250615232608.png]]
### flex-fill

- **功能**：讓元素擴展填滿剩餘空間
- **等同於**：CSS 的 `flex: 1 1 auto`
- **使用時機**：需要彈性寬度的元素

## 響應式變化

```html
<!-- 不同螢幕尺寸的 d-flex -->
<div class="d-sm-flex">  <!-- small 螢幕以上 -->
<div class="d-md-flex">  <!-- medium 螢幕以上 -->
<div class="d-lg-flex">  <!-- large 螢幕以上 -->
<div class="d-xl-flex">  <!-- extra large 螢幕以上 -->
```

## 實際開發場景

### 1. 表單欄位佈局

```html
<div class="d-flex align-items-center mb-3">
    <label class="me-3" style="min-width: 80px;">電子郵件：</label>
    <input type="email" class="form-control flex-fill">
    <button class="btn btn-outline-secondary ms-2">驗證</button>
</div>
```

### 2. 搜尋列

```html
<div class="d-flex">
    <select class="form-select me-2" style="max-width: 120px;">
        <option>全部</option>
        <option>標題</option>
    </select>
    <input type="text" class="form-control flex-fill" placeholder="請輸入關鍵字">
    <button class="btn btn-primary ms-2">搜尋</button>
</div>
```

### 3. 導航列佈局

```html
<nav class="d-flex">
    <div class="flex-fill">Logo</div>
    <div>選單項目</div>
</nav>
```

## Input Group 特殊情況

### 為什麼 flex-fill 要設定在 input-group 上？

**正確做法：**

```html
<div class="form-group form-inline">
    <div class="input-group flex-fill">        <!-- flex-fill 設定在這裡 -->
        <span class="input-group-text">https://</span>
        <input type="text" class="form-control" placeholder="網站名稱">
        <span class="input-group-text">.com</span>
    </div>
    <button class="btn btn-primary ml-1">搜尋</button>
</div>
```

**原因分析：**

1. **語義考量**：整個 input-group（前綴 + 輸入框 + 後綴）是一個完整的輸入單元
2. **佈局層級**：
    - 父容器：`form-group form-inline`
    - 子元素1：`input-group`（需要擴展）
    - 子元素2：`button`（固定寬度）
3. **避免破壞內部結構**：input-group 內部有自己的 flexbox 佈局機制

**錯誤做法（避免）：**

```html
<!-- 不要這樣做 -->
<div class="input-group">
    <span class="input-group-text">https://</span>
    <input type="text" class="form-control flex-fill">  <!-- 會破壞內部佈局 -->
    <span class="input-group-text">.com</span>
</div>
```

## 關鍵重點整理

1. **d-flex 只是啟用 Flexbox**，不會自動平均分配寬度
2. **需要 flex-fill 才會擴展**填滿剩餘空間
3. **通常只對需要彈性寬度的元素使用 flex-fill**（如 input、textarea）
4. **其他元素保持內容寬度**（如 label、button）
5. **Bootstrap 元件要考慮整體性**，flex-fill 設定在適當的容器層級上

## 實務建議

- 先確定哪個元素需要彈性寬度
- 考慮 Bootstrap 元件的完整性
- 善用 spacing utilities（如 me-2, ms-2）調整間距
- 使用 align-items-center 讓元素垂直置中對齊
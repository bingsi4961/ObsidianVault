---
date: 2025-06-27 22:44
aliases:
  - 別名測試1
  - 別名測試2
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

## 核心概念

### 影響範圍的差異

- **form-inline**：影響所有子元素和孫級元素（全部水平排列）
- **row**：只影響直接子元素（孫元素垂直排列保持不變）
- **form-row**：只影響直接子元素（孫元素垂直排列保持不變）

## 各類別詳細分析

### 1. form-inline

```css
.form-inline {
    display: flex;
    flex-flow: row wrap;
    align-items: center;
}

/* 關鍵：Bootstrap 額外定義子元素行為 */
.form-inline .form-control,
.form-inline .form-group,
.form-inline .input-group,
.form-inline .form-check {
    display: flex; /* 或 inline-block */
    align-items: center;
    margin-bottom: 0;
}

.form-inline label {
    margin-bottom: 0;
    margin-right: 0.5rem;
}
```

**特點：**

- 所有表單元素強制水平排列
- Bootstrap 定義了豐富的子元素樣式規則
- 適合簡單的單行表單

### 2. row

```css
.row {
    display: flex;
    flex-wrap: wrap;
    margin-right: -15px;  /* 標準 gutter */
    margin-left: -15px;
}
```

**特點：**

- 只影響直接子元素（通常是 col-* 類別）
- 15px 的標準間距
- 子元素內部保持垂直排列
- 通用的網格佈局容器

### 3. form-row

```css
.form-row {
    display: flex;
    flex-wrap: wrap;
    margin-right: -5px;   /* 較小的 gutter */
    margin-left: -5px;
}
```

**特點：**

- 只影響直接子元素
- 5px 的較小間距，專為表單設計
- 子元素內部保持垂直排列
- 專門用於表單的網格佈局

## 實際範例對比

### form-inline 範例

```html
<form class="form-inline">
    <div class="form-group">
        <label>姓名</label>                    <!-- 水平排列 -->
        <input type="text" class="form-control"> <!-- 水平排列 -->
        <small class="form-text">提示</small>    <!-- 水平排列 -->
    </div>
</form>
```

### row / form-row 範例

```html
<div class="form-row"> <!-- 或使用 row -->
    <div class="col-md-6">
        <label>姓名</label>                    <!-- 垂直排列 -->
        <input type="text" class="form-control"> <!-- 垂直排列 -->
        <small class="form-text">提示</small>    <!-- 垂直排列 -->
    </div>
    <div class="col-md-6">
        <label>電話</label>                    <!-- 垂直排列 -->
        <input type="text" class="form-control"> <!-- 垂直排列 -->
    </div>
</div>
```

## 關鍵差異總結

|類別|容器類型|影響範圍|間距|用途|
|---|---|---|---|---|
|form-inline|Flexbox|所有層級元素|-|單行水平表單|
|row|Flexbox|僅直接子元素|15px|通用網格佈局|
|form-row|Flexbox|僅直接子元素|5px|表單專用網格佈局|

## 除錯技巧

### 使用 F12 開發者工具查看樣式

1. **檢查元素**：右鍵 → 檢查元素
2. **查看 Computed 樣式**：切換到 "Computed" 分頁
3. **查看 Styles 面板**：在 "Styles" 分頁找到相關 CSS 規則

### 常見的 form-inline 子元素規則

```css
.form-inline .form-control {
    display: inline-block;
    width: auto;
    vertical-align: middle;
}

.form-inline .form-group {
    display: inline-block;
    margin-bottom: 0;
    vertical-align: middle;
}
```

## 使用建議

- **form-inline**：適用於搜尋框、登入表單等簡單的水平排列需求
- **row**：適用於一般的網格佈局，不限於表單
- **form-row**：適用於複雜表單的多欄位排列，間距較緊密適合表單元素
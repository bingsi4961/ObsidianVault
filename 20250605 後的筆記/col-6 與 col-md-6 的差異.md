---
date: 2025-06-28 13:34
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

#### 無斷點前綴表單 (col-\*)

```html
<div class="col-3">   <!-- 無前綴 -->
<div class="col-6">   <!-- 無前綴 -->
```

- **說明**：欄位不換行，只是寬度縮小
- **行為**：在所有螢幕尺寸下都保持固定比例
- **結果**：永遠不換行，只會縮小欄位寬度

#### 有斷點前綴表單 (col-md-\*)

```html
<div class="col-md-6">   <!-- 有 md 前綴 -->
```

- **說明**：欄位會自動換行
- **行為**：只在中等螢幕 (≥768px) 以上生效
- **結果**：小於 768px 時==自動變成 `col-12`，強制換行==

## Bootstrap 斷點系統

|斷點前綴|螢幕尺寸|說明|
|---|---|---|
|`col-`|所有尺寸|永遠生效，不響應式|
|`col-sm-`|≥576px|小螢幕以上|
|`col-md-`|≥768px|中等螢幕以上|
|`col-lg-`|≥992px|大螢幕以上|
|`col-xl-`|≥1200px|超大螢幕以上|

## 實際案例對比

### 無斷點前綴表單 (不會換行)

```html
<form>
    <div class="form-row">
        <div class="col-3">
            <input type="text" class="form-control" placeholder="名">
        </div>
        <div class="col-3">
            <input type="text" class="form-control" placeholder="姓">
        </div>
        <div class="col-6">
            <input type="text" class="form-control" placeholder="座右銘">
        </div>
    </div>
</form>
```

![[Pasted image 20250628135153.png]]

### 有斷點前綴表單 (會換行)

```html
<form>
    <div class="form-row">
        <div class="col-md-3">
            <input type="text" class="form-control" placeholder="名">
        </div>
        <div class="col-md-3">
            <input type="text" class="form-control" placeholder="姓">
        </div>
        <div class="col-md-6">
            <input type="text" class="form-control" placeholder="座右銘">
        </div>
    </div>
</form>
```

![[Pasted image 20250628135215.png]]

### 更精細的響應式控制

```html
<!-- 在不同螢幕尺寸有不同佈局 -->
<div class="col-12 col-sm-6 col-md-4 col-lg-3">
    <!-- 手機：佔滿整行 -->
    <!-- 小螢幕：一行兩個 -->
    <!-- 中螢幕：一行三個 -->
    <!-- 大螢幕：一行四個 -->
</div>
```

## 重點提醒

1. **無斷點前綴**：`col-3`, `col-6` → 永遠不換行
2. **有斷點前綴**：`col-md-6` → 小於斷點時會換行
3. **選擇原則**：
    - 需要響應式 → 使用斷點前綴
    - 固定比例 → 使用無前綴
4. **常用斷點**：`col-md-` 適用於大部分響應式需求 (768px)
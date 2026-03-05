---
date: 2026-03-05 10:35
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
# 連結筆記
#### 📑 [[]]

---

## 背景

在 Web 開發中，<mark style="background: #FFF3A3A6;">有時需要讓 Checkbox 保持勾選狀態且不可變更，同時又希望表單送出時，值能正常傳遞到後端</mark>。

然而 HTML 原生屬性無法完美達成這個需求：

- `readonly` 不支援 `checkbox` (加上去沒效果），主要用於 `text` 或 `textarea`。對於 `checkbox` 或 `radio`，使用者依然可以勾選或取消。
- `disabled` 可以鎖定，但**值不會隨表單送出**

利用 CSS `pointer-events: none` 搭配 `tabindex="-1"`，可以同時滿足「鎖定交互」與「保留表單值」。

---

## 實作範例

### CSS

```css
/* 鎖定已勾選的 checkbox，不允許變動 */
.locked-checkbox {
    pointer-events: none;  /* 核心：讓滑鼠事件「穿透」該元素，點擊無效 */
    opacity: 0.6;          /* 視覺提示：讓使用者知道此項目前不可變更 */
}
```

### HTML

```html
<div class="form-check">
    <input type="checkbox"
           value="1"
           class="form-check-input locked-checkbox"
           onclick="toggleSkuItemAll(this);"
           checked=""
           tabindex="-1">
    <label class="form-check-label lbl-notify-all locked-checkbox">All</label>
</div>
```

### 關鍵屬性說明

|屬性|作用|
|---|---|
|`pointer-events: none`|滑鼠點擊事件穿透該元素，使用者無法點擊|
|`opacity: 0.6`|半透明外觀，暗示該欄位目前不可操作|
|`tabindex="-1"`|防止使用者透過 `Tab` 鍵聚焦後按空白鍵改變狀態|

---

## 三種方案比較

| 特性    | `disabled` | `readonly`（Checkbox） | `pointer-events: none` |
| ----- | ---------- | -------------------- | ---------------------- |
| 滑鼠點擊  | 禁止         | **允許**               | 禁止                     |
| 表單送出值 | **不會送出**   | 會送出                  | **會送出**                |
| 鍵盤聚焦  | 無法聚焦       | 可聚焦                  | 需搭配 `tabindex="-1"`    |
| 適用場景  | 完全不處理該欄位   | 不適用於 Checkbox        | 需保留值但不讓用戶修改            |

---

## 潛在風險與注意事項

### 1. F12 開發者工具可繞過

進階使用者可透過瀏覽器開發者工具移除 CSS class，進而改變勾選狀態。

> **重點：後端 Controller 仍需驗證該欄位是否允許變動，不可僅依賴前端鎖定。**

### 2. jQuery 程式碼觸發不受影響

`pointer-events: none` 只擋得住**滑鼠物理點擊**。若透過 JavaScript / jQuery 程式碼操作：

```javascript
$('.form-check-input').prop('checked', true);
```

仍然可以改變 Checkbox 狀態，不會被 `pointer-events` 阻擋。

### 3. 瀏覽器相容性

現代瀏覽器（Chrome、Edge、Firefox、Safari）對 `pointer-events` 的支援度已非常成熟，在 Bootstrap 4.3.1（Portal）及 Bootstrap 3.0.0（GTS）環境下皆可正常運作。

---

## 適用情境

- 表單中有些 Checkbox 根據業務邏輯必須維持勾選，但值仍需送出
- 顯示「已鎖定」的項目清單，讓使用者知道這些選項不可變更
- 搭配 jQuery 動態 ( addClass、removeClass ) 切換 `.locked-checkbox` class，實現條件式鎖定
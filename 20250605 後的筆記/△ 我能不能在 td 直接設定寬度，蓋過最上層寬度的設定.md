---
date: 2025-06-18 09:54
aliases: 
tags:
  - 入庫
  - CSS
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

內文開始

讓我解釋為什麼單純設定 td 的寬度可能不夠，以及如何正確地實現這個目標。

直接在 td 上設定寬度確實是一種方法，但我們需要注意一些重要的細節。想像一下一個俄羅斯套娃的結構：即使最小的娃娃（td）想要維持特定的大小，如果外層的娃娃（容器）強制施加了限制，內層還是會被擠壓。

要讓 td 的寬度設定真正生效，我們需要：

```css
#PlmsTable td[data-db-table="project"],
#PlmsTable td[data-db-table="list"] {
    min-width: 250px !important;     /* 使用 !important 來確保優先權 */
    width: 250px !important;         /* 同時設定 width 和 min-width */
    max-width: 250px !important;     /* 設定 max-width 防止過度延展 */
    box-sizing: border-box;          /* 確保 padding 不會影響設定的寬度 */
}

/* 特別處理標題列 */
#PlmsTable th.RightBorder {
    min-width: 150px !important;
    width: 150px !important;
    max-width: 150px !important;
    box-sizing: border-box;
}
```

為什麼需要這麼多設定？讓我解釋每個屬性的作用：

1. `!important` 關鍵字
   - 這會提高樣式的優先級
   - 幫助覆蓋來自父層容器的寬度限制
   - 但要小心使用，因為過度使用會讓樣式難以維護

2. 同時設定三種寬度屬性
   - `width`: 設定基本寬度
   - `min-width`: 確保不會被壓縮得更小
   - `max-width`: 防止過度延展

3. `box-sizing: border-box`
   - 確保 padding 和 border 都包含在設定的寬度內
   - 防止實際寬度超出預期

這種方法的效果相當於告訴瀏覽器："不管外層容器如何設定，這個單元格的寬度都必須保持這個大小。"不過，要讓這個方法完全發揮作用，建議再加上一層保險：

```css
/* 確保表格本身也支持內容延展 */
#PlmsTable {
    table-layout: fixed;       /* 強制表格遵守我們設定的寬度 */
    width: auto !important;    /* 允許表格依據內容延展 */
    white-space: normal;       /* 允許內容換行 */
}

/* 確保內容不會溢出 */
#PlmsTable td .form-control,
#PlmsTable td .input-group {
    width: 100%;              /* 限制內部元素寬度 */
    overflow: hidden;         /* 防止內容溢出 */
}
```

這樣的設定組合能夠：
- 有效對抗上層容器的寬度限制
- 保持表格欄位的一致性
- 確保內容的可讀性
- 防止版面破裂

你可以根據實際需求調整具體的寬度數值。需要我更詳細解釋任何部分嗎？或者你想了解如何處理特定的內容顯示問題？
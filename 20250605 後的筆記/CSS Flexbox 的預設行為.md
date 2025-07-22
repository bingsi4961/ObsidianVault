---
date: 2025-06-26 11:42
aliases: 
tags:
  - CSS
  - Flexbox
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

```css
.flex-container {
    display: flex;    
    /* 以下是瀏覽器自動套用的預設值 */
    flex-direction: row;          /* 主軸方向：水平從左到右 */
    flex-wrap: nowrap;            /* 不換行 */
    justify-content: flex-start;  /* 主軸對齊：靠開始位置 */
    align-items: stretch;         /* 交叉軸對齊：stretch 行為 */
    
    align-content: flex-start;    /* 當子元素有固定高度時 */
    align-content: stretch;       /* 當子元素沒有固定高度時 */
    
    gap: 0rem;                    /* 子元素之間的間距：0 */
}
```
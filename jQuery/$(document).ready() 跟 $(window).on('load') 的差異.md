---
title: $(document).ready() 跟 $(window).on('load') 的差異
tags: [jQuery]

---

`$(document).ready()` 確實是在 DOM 結構解析完成後觸發的，但它僅表示 HTML 文檔結構已經可用，而不是所有資源都加載完成。具體來說：

1. `$(document).ready()`：在 DOM 結構(HTML元素樹)解析完成後立即執行，此時：
   - 所有 HTML 元素已經可以被 JavaScript 選取和操作
   - 但圖片、樣式表、iframe 等外部資源可能尚未完全加載完成

2. `$(window).on('load')`：在頁面上所有元素（包括圖片、樣式、iframe等）都完全加載完成後執行

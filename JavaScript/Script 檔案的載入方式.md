---
title: Script 檔案的載入方式
tags: [JavaScript]

---

1. 一般載入 (無屬性)：
```html
<script src="spmcost.js"></script>
```
- HTML 解析遇到 script 標籤時會暫停
- 下載並執行 JavaScript 檔案
- 完成後才繼續解析 HTML
- 會阻塞頁面渲染
- js 檔案會照順序執行

2. async 載入：
```html
<script src="spmcost.js" async></script>
```
- HTML 解析不會暫停，繼續進行
- JavaScript 檔案會同時下載
- 下載完成後立即執行
- ==不保證執行順序 (誰先下載完就先執行誰)==
- 適合獨立的腳本，如 Google Analytics

3. defer 載入：
```html
<script src="spmcost.js" defer></script>
```
- HTML 解析不會暫停，繼續進行
- JavaScript 檔案會同時下載
- ==等 HTML 解析完成後才執行==
- ==保證執行順序 (按照 script 標籤的順序執行)==
- ==適合需要操作 DOM 或依賴其他腳本的情況==


---
1. jquery.min.js (核心庫)
2. jquery.validate.min.js (依賴於 jQuery)
3. additional-methods.min.js (依賴於 jQuery Validate)
4. jquery.validate.unobtrusive.min.js (依賴於 jQuery Validate)

如果都使用 `defer`，應該這樣寫：

```html
<script src="jquery.min.js" defer></script>
<script src="jquery.validate.min.js" defer></script>
<script src="additional-methods.min.js" defer></script>
<script src="jquery.validate.unobtrusive.min.js" defer></script>
```

使用 `defer` 是安全的，因為：
1. defer 會按照 script 標籤的順序執行
2. 所有 defer 的腳本會等到 HTML 解析完才執行
3. 雖然檔案會同時下載，但執行順序會保持正確

相比之下，如果使用 `async`：
```html
<script src="jquery.min.js" async></script>
<script src="jquery.validate.min.js" async></script>
<script src="additional-methods.min.js" async></script>
<script src="jquery.validate.unobtrusive.min.js" async></script>
```
這樣會有問題，因為：
1. async 的腳本下載完就會立即執行
2. 不保證執行順序
3. 可能會造成相依性錯誤

所以：
==★ 對於有相依性的腳本，使用 defer==
==★ 對於獨立的腳本（如 Google Analytics），可以使用 async==
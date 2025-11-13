---
date: 2025-11-06 10:22
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
```javascript
	$row.addClass('row-editing');
	$row.removeClass('row-editing');
```

```css
.row-editing {
	background-color: #e7f3ff !important;
	border-left: 2px solid ##5a9fd4;    /* 左側加上淡藍色邊框 */
}

	.row-editing td {
		background-color: #e7f3ff !important;
		padding: 8px 8px;
	}
	
	.row-editing input,
	.row-editing select {
		background-color: #ffffff;
		border: 1px solid #5a9fd4;
	}
```

```css
.modal-body > select {
	margin-bottom: 3px;
}
```

```css
.file-column { 
	max-width: 150px;
	
	/* word-wrap: break-word; 
	  (現在的標準屬性名稱是 overflow-wrap: break-word; 但 word-wrap 相容性更好) 
	*/
	word-wrap: break-word;     
	/* 防止「單一且過長」的單字或 URL 撐破容器。
		策略：先禮後兵。
		1. 瀏覽器會優先嘗試在正常的空白處 (空格) 換行。
		2. 只有當一整行都沒有可換行的空白 (例如一個超長單字)，
		   它才會在那個「長單字」的中間強行斷開，讓它換到下一行。
		3. 對於正常的句子排版破壞性最小。
	*/

	word-break: break-all;      
	/* 強制「所有文字」在碰到容器邊界時換行。
	    策略：一視同仁，全部切斷。
	    1. 它不考慮單字的完整性，也不會優先去找空格。
		2. 只要文字碰到邊界，就立刻在那個字元處斷開 (例如 "Hello" 可能被斷成 "Hel" 和 "lo")。
		3. 在中、日、韓文 (CJK) 語言中很常用 (因為它們本來就不是靠空格斷詞)，但在英文環境下會顯得排版很破碎。
	*/

	white-space: normal;        
	/* 這是瀏覽器處理空白和換行的「預設行為」。
		1. 合併空白：原始碼中連續的多個空格或 Tab 鍵，在畫面上會被壓縮成一個空格。
		2. 自動換行：文字碰到容器邊緣時，會自動在「空白字元」(空格) 處換到下一行。
		3. 忽略換行：原始碼中的換行符 (Enter) 會被當成一個普通的空格處理。
		(您寫的「允許多行顯示」是正確的結果，這裡是說明它如何達成多行)
	*/
}
```
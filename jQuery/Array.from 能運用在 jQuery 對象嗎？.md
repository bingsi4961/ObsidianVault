---
title: Array.from 能運用在 jQuery 對象嗎？
tags: [JavaScript, jQuery]

---

是的，Array.from 絕對可以運用在 jQuery 對象上，而且這是一個非常實用的技巧。這是因為 jQuery 對象本身就是一種類數組對象（array-like object）。

## jQuery 對象的類數組性質

當你創建一個 jQuery 對象時（例如 `$('div')`），它實際上會：

1. 包含所有匹配的 DOM 元素，每個元素都存儲在以 0 開始的數字索引中
2. 擁有一個 `length` 屬性，表示匹配元素的數量
3. 提供類似數組的遍歷方法，如 `.each()`

這些特性使 jQuery 對象符合類數組對象的定義。

## 使用 Array.from 轉換 jQuery 對象的實例

假設你的頁面上有三個段落元素：

```html
<p class="highlight">第一段</p>
<p>第二段</p>
<p class="highlight">第三段</p>
```

你可以用 jQuery 選取它們，然後用 Array.from 轉換：

```javascript
// 選取所有段落元素
const $paragraphs = $('p');
console.log($paragraphs.length); // 3

// 將 jQuery 對象轉換為真正的數組
const paragraphsArray = Array.from($paragraphs);
console.log(paragraphsArray); // [p元素, p元素, p元素]

// 現在你可以使用真正數組的所有方法
const highlightedParagraphs = paragraphsArray.filter(element => 
  element.classList.contains('highlight')
);
console.log(highlightedParagraphs.length); // 2
```

## 為什麼要將 jQuery 對象轉換為數組？

將 jQuery 對象轉換為真正的數組有幾個重要優勢：

1. **使用現代數組方法**：獲得 `map()`, `filter()`, `reduce()`, `find()` 等所有現代 JavaScript 數組方法的便利性

2. **更簡潔的語法**：比起 jQuery 的鏈式方法，原生數組方法通常提供更簡潔的語法

3. **更好的性能**：原生數組方法通常比 jQuery 等效方法執行得更快

4. **混合使用 jQuery 和現代 JavaScript**：允許在使用 jQuery 處理 DOM 的同時利用現代 JavaScript 特性

## 實用示例：映射和篩選 jQuery 結果

假設你想獲取頁面上所有具有特定數據屬性的圖像，並提取它們的 URL：

```javascript
// jQuery 方式
const imageUrls = [];
$('img[data-category="product"]').each(function() {
  imageUrls.push($(this).attr('src'));
});

// 使用 Array.from 的方式
const images = $('img[data-category="product"]');
const imageUrls = Array.from(images).map(img => img.src);
```

## 結合 Array.from 的第二個參數（映射函數）

Array.from 接受第二個參數作為映射函數，這在處理 jQuery 對象時特別有用：

```javascript
// 選取所有輸入框並提取它們的值
const $inputs = $('input');
const inputValues = Array.from($inputs, input => input.value);

// 獲取所有元素的數據屬性
const $elements = $('.data-item');
const dataIds = Array.from($elements, el => $(el).data('id'));
```

## 深入理解：jQuery 對象的內部結構為何適合 Array.from

jQuery 對象的內部結構很像一個類數組對象：

```javascript
// 簡化的 jQuery 對象結構
{
  0: DOMElement,
  1: DOMElement,
  2: DOMElement,
  length: 3,
  // ...jQuery 的各種方法和屬性
}
```

這種結構完全符合 Array.from 的要求：擁有 0-based 數字索引和 length 屬性。

## 實際場景：處理表單元素集合

假設你有一個表單，需要收集所有輸入框的值：

```javascript
// 假設頁面上有一個表單
const $form = $('#myForm');
const $inputs = $form.find('input, select, textarea');

// 收集所有值並過濾掉空值
const formValues = Array.from($inputs)
  .filter(input => input.value.trim() !== '')
  .map(input => ({
    name: input.name,
    value: input.value,
    type: input.type
  }));

console.log(formValues);
// [{name: "username", value: "Ethan", type: "text"}, ...]
```

這種方法比使用 jQuery 的 `.each()` 方法更簡潔，更容易理解，並且可以利用更多現代 JavaScript 數組方法的優勢。
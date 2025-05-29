---
title: filter
tags: [jQuery]

---

jQuery 的 filter 方法主要用於從已選取的元素集合中篩選出符合特定條件的元素。
讓我為您詳細說明幾種常用的 filter 方法：

1. 基本選擇器過濾：

```javascript
// 選擇所有 <p> 中 class 為 "highlight" 的元素
$("p").filter(".highlight")

// 選擇所有 <div> 中 id 為 "special" 的元素
$("div").filter("#special")
```

2. 使用函數過濾：

```javascript
// 選擇索引為偶數的 <li> 元素
$("li").filter(function(index) {
    return index % 2 === 0;
})

// 選擇內容包含 "Hello" 的元素
$("p").filter(function() {
    return $(this).text().includes("Hello");
})
```

3. 使用 :first 和 :last 過濾器：

```javascript
// 選擇第一個 <div> 元素
$("div").filter(":first")

// 選擇最後一個 <p> 元素
$("p").filter(":last")
```

實際的應用例子：

```javascript
// HTML 結構
// <ul>
//   <li>項目 1</li>
//   <li class="important">項目 2</li>
//   <li>項目 3</li>
//   <li class="important">項目 4</li>
// </ul>

// 為所有 class 為 "important" 的 <li> 添加紅色背景
$("li").filter(".important").css("background-color", "red");

// 選擇內容長度大於 5 的元素
$("li").filter(function() {
    return $(this).text().length > 5;
}).addClass("long-text");
```

filter 方法的一些重要特點：

1. 鏈式操作：filter 可以與其他 jQuery 方法串連使用
2. 回傳值：filter 會返回一個新的 jQuery 物件
3. 靈活性：可以使用選擇器、函數或 jQuery 表達式作為過濾條件

---

### 讓我解釋 jQuery filter 返回新物件的概念。

當我們說 filter 返回一個新的 jQuery 物件時，意思是 filter 不會修改原始的選擇結果，而是創建一個包含過濾後元素的全新 jQuery 物件。讓我用實例說明：

```javascript
// 假設我們有這樣的 HTML：
// <ul>
//   <li class="fruit">蘋果</li>
//   <li class="fruit">香蕉</li>
//   <li class="vegetable">胡蘿蔔</li>
//   <li class="fruit">橘子</li>
// </ul>

// 先選取所有 li 元素
var allItems = $("li");             // 包含 4 個 li 元素
console.log(allItems.length);       // 輸出：4

// 使用 filter 過濾出 fruit 類別
var fruitItems = allItems.filter(".fruit");  // 新物件，只包含 3 個 fruit 元素
console.log(fruitItems.length);     // 輸出：3

// 原始的 allItems 物件不受影響
console.log(allItems.length);       // 仍然是 4
```

這個概念很重要，因為：

1. **原始選擇保持不變**：
```javascript
var allLinks = $("a");              // 選取所有連結
var activeLinks = allLinks.filter(".active");  // 過濾出啟用的連結
// allLinks 仍然包含所有連結
// activeLinks 是新物件，只包含啟用的連結
```

2. **可以繼續鏈式操作**：
```javascript
$("li")
    .filter(".fruit")               // 返回新物件，只包含水果
    .css("color", "red")           // 對過濾後的結果操作
    .addClass("selected");         // 繼續操作同一組過濾後的元素
```

3. **可以保存不同的結果集**：
```javascript
var allProducts = $(".product");    // 所有產品
var inStock = allProducts.filter(".in-stock");    // 有庫存的產品
var outOfStock = allProducts.filter(".out-of-stock"); // 缺貨的產品

// 可以分別操作這些不同的結果集
inStock.css("color", "green");
outOfStock.css("color", "red");
```
---
雖然 filter 會返回新的 jQuery 物件，但這些物件都是參考到相同的 DOM 元素。
```javascript
// HTML 結構
// <div class="product in-stock">商品 1</div>
// <div class="product out-of-stock">商品 2</div>
// <div class="product in-stock">商品 3</div>

// 選取所有產品
var allProducts = $(".product");

// 過濾出有庫存的產品
var inStock = allProducts.filter(".in-stock");

// 設定有庫存產品的顏色
inStock.css("color", "green");

// 這時候檢查 allProducts 中的元素
console.log($(".product.in-stock").css("color")); // 會顯示 "green"
```

所以：
1. 雖然 `inStock` 是新的 jQuery 物件
2. 但它參考的 DOM 元素和 `allProducts` 中的相同元素是同一個
3. 當您修改 `inStock` 中元素的 CSS，實際上是修改了 DOM 元素
4. 因此，無論您通過哪個 jQuery 物件去查看這個元素，都會看到更新後的樣式

這就像是：
- 您有一個房間（DOM 元素）
- `allProducts` 和 `inStock` 就像是兩個不同的門（jQuery 物件）
- 這兩個門都通向同一個房間
- 如果您通過其中一個門進去粉刷牆壁（修改 CSS）
- 當您從另一個門進去看時，牆壁的顏色也是改變了的

要驗證這一點，您可以試試：

```javascript
// 選取所有產品
var allProducts = $(".product");
console.log("原始數量:", allProducts.length); // 例如：3

// 過濾有庫存商品
var inStock = allProducts.filter(".in-stock");
console.log("有庫存數量:", inStock.length); // 例如：2

// 修改有庫存商品的樣式
inStock.css("color", "green");

// 檢查所有商品中有庫存的元素的顏色
allProducts.filter(".in-stock").each(function() {
    console.log("顏色:", $(this).css("color")); // 會顯示 "green"
});
```

這是因為 jQuery 物件是對 DOM 元素的引用集合，而不是元素本身的複製品。當您使用 filter 時，您創建了一個新的 jQuery 物件，但這個物件仍然參考到原始的 DOM 元素。
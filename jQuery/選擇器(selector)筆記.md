---
title: 選擇器(selector)筆記
tags: [jQuery]

---

# 選擇器(selector)筆記

## 一、基本選擇器 (Basic Selectors)

### 1.1 元素選擇器
```javascript
$('div')         // 選擇所有 div 元素
$('p')           // 選擇所有段落元素
$('a')           // 選擇所有連結
```

### 1.2 ID 選擇器
```javascript
$('#header')     // 選擇 ID 為 "header" 的元素
$('#content')    // 選擇 ID 為 "content" 的元素
```

### 1.3 Class 選擇器
```javascript
$('.item')       // 選擇所有 class 為 "item" 的元素
$('.highlight')  // 選擇所有 class 為 "highlight" 的元素
```

### 1.4 組合選擇器
```javascript
$('div.container')  // 選擇 class 為 "container" 的 div 元素
$('p.text')         // 選擇 class 為 "text" 的段落
$('#menu a')        // 選擇 ID 為 "menu" 的元素中的所有連結
```

### 1.5 多重選擇器
```javascript
$('h1, h2, h3')     // 選擇所有 h1, h2 和 h3 元素
$('div, p, span')   // 選擇所有 div, p 和 span 元素
$('#header, .footer') // 選擇 ID 為 "header" 的元素和 class 為 "footer" 的元素
```

## 二、層次選擇器 (Hierarchy Selectors)

### 2.1 後代選擇器（空格）
選擇指定元素內所有符合條件的後代元素（包括子元素、孫元素等所有層級）。
```javascript
$('#container p')         // 選擇 #container 內的所有 p 元素(所有後代)
$('.article span')        // 選擇 .article 內的所有 span 元素
$('div.container ul li a')  // 多層後代關係
```

### 2.2 子元素選擇器（>）
只選擇直接子元素，不包括更深層級的後代元素。
```javascript
$('#nav > li')            // 選擇 #nav 的直接子元素中的 li 元素
$('.box > p')             // 選擇 .box 的直接子元素中的 p 元素
$('ul > li a')            // 混合使用子元素和後代選擇器
```

### 2.3 相鄰兄弟選擇器（+）
選擇緊接在指定元素後的兄弟元素（==僅限單個元素==）。
```javascript
$('h2 + p')               // 選擇緊接在 h2 後面的 p 元素
$('.title + div')         // 選擇緊接在 .title 後面的 div 元素
$('#mainSection > h3 + p')  // 與其他選擇器組合使用
```

### 2.4 一般兄弟選擇器（~）
選擇指定元素後的所有兄弟元素（==同級且在後面的所有元素==）。
```javascript
$('h2 ~ p')               // 選擇 h2 後面的所有 p 兄弟元素
$('.header ~ .item')      // 選擇 .header 後面的所有 .item 兄弟元素
$('h1 ~ p:contains("important")')  // 與過濾選擇器組合
```

## 三、偽類選擇器 (Pseudo-class Selectors)

### 3.1 狀態偽類
```javascript
$('a:hover')        // 選擇懸停狀態的連結（在 jQuery 中有限制）
$('a:visited')      // 選擇已訪問的連結（在 jQuery 中有限制）
$('input:focus')    // 選擇有焦點的輸入框
$('input:checked')  // 選擇所有被選中的 checkbox 或 radio
$('input:enabled')  // 選擇啟用的輸入元素
$('input:disabled') // 選擇禁用的輸入元素
```

### 3.2 位置偽類
```javascript
$('li:first')          // 選擇第一個 li 元素
$('li:last')           // 選擇最後一個 li 元素
$('li:odd')            // 選擇索引為奇數的 li 元素（索引從 0 開始）
$('li:even')           // 選擇索引為偶數的 li 元素
$('tr:nth-child(3)')   // 選擇第 3 個子元素是 tr 的元素
$('p:nth-of-type(2)')  // 選擇第 2 個 p 元素
$('li:lt(3)')          // 選擇前 3 個 li 元素（索引 0, 1, 2）
$('li:gt(2)')          // 選擇索引大於 2 的 li 元素
$('li:eq(2)')          // 選擇索引為 2 的 li 元素（第 3 個元素）
```

### 3.3 內容偽類
```javascript
$('div:empty')             // 選擇沒有子元素的 div
$('p:contains("Hello")')   // 選擇包含 "Hello" 文本的段落
$('li:has(a)')             // 選擇包含 a 元素的 li 元素
$('li:parent')             // 選擇有子元素或文本的 li 元素（非空元素）
```

### 3.4 表單偽類
```javascript
$('input:text')     // 選擇 type 為 text 的輸入框
$('input:password') // 選擇 type 為 password 的輸入框
$('input:radio')    // 選擇 type 為 radio 的輸入框
$('input:checkbox') // 選擇 type 為 checkbox 的輸入框
$('input:button')   // 選擇 type 為 button 的輸入框
$('input:submit')   // 選擇 type 為 submit 的輸入框
$('input:reset')    // 選擇 type 為 reset 的輸入框
$('input:file')     // 選擇 type 為 file 的輸入框
$('input:hidden')   // 選擇隱藏的輸入元素
```

### 3.5 其他偽類
```javascript
$(':animated')      // 選擇所有正在執行動畫的元素
$(':visible')       // 選擇所有可見的元素
$(':hidden')        // 選擇所有隱藏的元素(不只有input hidden)
$(':not(selector)') // 選擇不符合指定選擇器的元素
```

## 四、屬性選擇器 (Attribute Selectors)

### 4.1 基本屬性選擇器
```javascript
$('a[href]')        // 選擇有 href 屬性的連結
$('input[type]')    // 選擇有 type 屬性的輸入框
$('img[alt]')       // 選擇有 alt 屬性的圖片
```

### 4.2 屬性值選擇器
```javascript
$('a[href="https://example.com"]') // 選擇 href 屬性等於 "https://example.com" 的連結
$('input[type="text"]')            // 選擇 type 屬性為 "text" 的輸入框
```

### 4.3 屬性值包含選擇器(==不需要完整的單詞==)
```javascript
$('a[href*="example"]')      // 選擇 href 屬性中包含 "example" 的連結
$('div[class*="container"]') // 選擇 class 屬性中包含 "container" 的 div
```

### 4.4 屬性值開頭選擇器
```javascript
$('a[href^="https"]')      // 選擇 href 屬性以 "https" 開頭的連結
$('input[name^="user"]')   // 選擇 name 屬性以 "user" 開頭的輸入框
```

### 4.5 屬性值結尾選擇器
```javascript
$('a[href$=".pdf"]')       // 選擇 href 屬性以 ".pdf" 結尾的連結
$('img[src$=".jpg"]')      // 選擇 src 屬性以 ".jpg" 結尾的圖片
```

### 4.6 屬性值單詞選擇器(==需要完整的單詞== / 以「空白格、開頭、結尾」為字詞邊界)
```javascript
$('div[class~="main"]')    // 選擇 class 屬性中包含單詞 "main" 的 div
```

### 4.7 屬性值前綴選擇器
```javascript
$('div[lang|="en"]')       // 選擇 lang 屬性為 "en" 或以 "en-" 開頭的 div
```

### 4.8 屬性值不等於選擇器
```javascript
$('input[type!="checkbox"]') // 選擇 type 屬性不等於 "checkbox" 的輸入框
```

### 4.9 多重屬性選擇器
```javascript
$('a[href][target]')                         // 選擇同時擁有 href 和 target 屬性的 a 元素
$('input[type="text"][name*="user"]')        // 選擇 type 為 text 且 name 包含 user 的輸入框
$('img[src$=".jpg"][alt]')                   // 選擇 src 以 .jpg 結尾且有 alt 屬性的圖片
$('button.btn[data-toggle][data-target]')    // 結合類選擇器和多重屬性選擇器
```

## 五、特殊選擇器詳解

### 5.1 :not(selector) 選擇器
否定選擇器，用於選擇不符合指定選擇器的元素。

```javascript
$(':not(p)')                    // 選擇所有不是段落的元素
$('li:not(.active)')            // 選擇沒有 "active" 類的 li 元素
$('input:not(:disabled)')       // 選擇不是禁用的輸入框
$(':not(.nav):not(div)')        // 選擇所有不包含 "nav" 類且不是 div 的元素
$('li:not(:has(a))')            // 選擇不包含連結的列表項
```

### 5.2 :has(selector) 選擇器
選擇包含指定選擇器匹配元素的元素。

```javascript
$('div:has(p)')                 // 選擇包含 p 元素的 div
$('a:has(img)')                 // 選擇包含圖片的連結
$('li:has(input:checkbox:checked)') // 選擇包含選中的 checkbox 的 li 元素
$('div:has(p:contains("Hello"))') // 選擇包含帶有特定文本的段落的 div
```

### 5.3 :parent 選擇器
選擇有子元素或文本的元素（非空元素）。

```javascript
$('li:parent')                  // 選擇所有非空的 li 元素
$('div.container:parent')       // 選擇有內容且帶有特定類的 div
$('div:hidden:parent')          // 選擇有內容的隱藏元素
```

### 5.4 ==:hidden 選擇器==
選擇所有隱藏的元素。元素在以下情況被視為隱藏：
- CSS `display: none`
- `type="hidden"` 的表單元素
- 寬度和高度都設為 0
- 父元素是隱藏的（隱藏狀態會繼承）

```javascript
$(':hidden')                    // 選擇所有隱藏的元素
$('input:hidden')               // 選擇所有隱藏的輸入框
$('div.content:hidden')         // 選擇所有隱藏的且有 "content" 類的 div
$(':hidden[data-secret]')       // 選擇隱藏的且有特定屬性的元素
```

## 六、元素位置方法

### 6.1 eq() 方法
從匹配的元素集合中選取特定索引位置的元素。

```javascript
$('p').eq(2)                    // 選擇匹配的第三個段落元素（索引為 2）
$('li').eq(-1)                  // 選擇最後一個 li 元素（使用負索引）
```

### 6.2 first() 方法
返回匹配元素集合中的第一個元素。

```javascript
$('p').first()                  // 選擇第一個段落
$('.item').first()              // 選擇第一個帶有特定類的元素
$('form input').first().focus() // 處理第一個表單元素
```

### 6.3 last() 方法
返回匹配元素集合中的最後一個元素。

```javascript
$('p').last()                   // 選擇最後一個段落
$('li').last()                  // 選擇最後一個列表項
$('.pricing-plan').last().addClass('most-popular') // 為最後一個項目添加特殊樣式
```

### 6.4 index() 方法
[jQuery index() 方法完整筆記]([4lUZa5wATBiuIJsptbUy6Q](https://hackmd.io/4lUZa5wATBiuIJsptbUy6Q))
確定元素在集合中的位置或者在兄弟元素中的位置。
```javascript
// 無參數 - 獲取元素在其兄弟元素中的位置
$('li.active').index()  

// 帶選擇器 - 獲取元素在指定選擇器匹配的集合中的索引
var index = $('div').index($('#selected'));

// 在事件處理中使用
$('ul li').click(function() {
  var clickedIndex = $(this).index();
  console.log('您點擊了第 ' + (clickedIndex + 1) + ' 個項目');
});
```

## 七、效能優化考量

### 7.1 選擇器效率
- ID 選擇器最快（`#id`）
- 類選擇器次之（`.class`）
- 元素選擇器再次之（`div`）
- 偽類選擇器和屬性選擇器較慢（`:hidden`, `[attribute]`）

### 7.2 優化技巧
1. **縮小選擇範圍**
   ```javascript
   // 較慢
   $('div.product-item');
   
   // 較快（如果 .product-item 是唯一的）
   $('.product-item');
   ```

2. **從右到左優化**
   ```javascript
   // 較低效率
   $('#container div.special');
   
   // 較高效率
   $('.special');  // 如果 .special 類是唯一的
   ```

3. **避免過深的選擇器鏈**
   ```javascript
   // 避免
   $('body > div > article > section > div > p');
   
   // 更好的方法
   $('.content-section p');
   ```

4. **使用 find() 方法替代複雜選擇器**
   ```javascript
   // 較高效
   $('#container').find('.items');
   
   // 而不是
   $('#container .items');
   ```

5. **快取選擇器結果**
   ```javascript
   // 快取 jQuery 對象
   var $items = $('.item');
   
   // 重複使用
   $items.first().addClass('first');
   $items.last().addClass('last');
   ```

### 7.3 選擇器和方法替代方案
```javascript
// 選擇器替代方案
$('li:eq(2)')      等同於      $('li').eq(2)
$('li:first')      等同於      $('li').first()
$('li:last')       等同於      $('li').last()
$(':hidden')       等同於      $('*').filter(':hidden')
$('div:has(p)')    等同於      $('div').has('p')
```

---

## 八、參考速查表

### 基本選擇器
| 選擇器         | 說明                 | 示例                |
|--------------|---------------------|---------------------|
| `#id`        | ID 選擇器             | `$('#header')`      |
| `.class`     | Class 選擇器          | `$('.item')`        |
| `element`    | 元素選擇器             | `$('div')`          |
| `*`          | 全部選擇器             | `$('*')`            |
| `selector1, selector2` | 多重選擇器   | `$('h1, h2, h3')`   |

### 層次選擇器
| 選擇器         | 說明                 | 示例                     |
|--------------|---------------------|--------------------------|
| `parent descendant` | 後代選擇器      | `$('#nav a')`            |
| `parent > child`    | 子元素選擇器    | `$('ul > li')`           |
| `prev + next`       | 相鄰兄弟選擇器  | `$('h2 + p')`            |
| `prev ~ siblings`   | 一般兄弟選擇器  | `$('h2 ~ p')`            |

### 屬性選擇器
| 選擇器              | 說明                       | 示例                         |
|-------------------|----------------------------|------------------------------|
| `[attr]`          | 擁有屬性                    | `$('a[href]')`               |
| `[attr=value]`    | 屬性等於值                  | `$('input[type="text"]')`    |
| `[attr!=value]`   | 屬性不等於值                | `$('input[type!="checkbox"]')` |
| `[attr^=value]`   | 屬性值以指定值開頭           | `$('a[href^="https"]')`      |
| `[attr$=value]`   | 屬性值以指定值結尾           | `$('a[href$=".pdf"]')`       |
| `[attr*=value]`   | 屬性值包含指定值            | `$('a[href*="example"]')`    |
| `[attr~=value]`   | 屬性值包含指定單詞          | `$('div[class~="main"]')`    |
| `[attr][attr2]`   | 多重屬性選擇器              | `$('input[type][name]')`     |

### 位置選擇器和方法
| 選擇器/方法         | 說明                       | 示例                      |
|-------------------|----------------------------|---------------------------|
| `:first`          | 第一個元素                  | `$('li:first')`           |
| `:last`           | 最後一個元素                | `$('li:last')`            |
| `:eq(n)`          | 索引等於 n 的元素           | `$('li:eq(2)')`           |
| `:gt(n)`          | 索引大於 n 的元素           | `$('li:gt(2)')`           |
| `:lt(n)`          | 索引小於 n 的元素           | `$('li:lt(2)')`           |
| `.eq(n)`          | 選擇索引為 n 的元素         | `$('li').eq(2)`           |
| `.first()`        | 選擇第一個元素              | `$('li').first()`         |
| `.last()`         | 選擇最後一個元素            | `$('li').last()`          |
| `.index()`        | 獲取元素索引                | `$('li.active').index()`  |

### 內容與可見性選擇器
| 選擇器              | 說明                       | 示例                         |
|-------------------|----------------------------|------------------------------|
| `:contains(text)` | 包含指定文本的元素          | `$('p:contains("Hello")')`   |
| `:empty`          | 沒有子元素或文本的元素      | `$('div:empty')`             |
| `:parent`         | 有子元素或文本的元素        | `$('li:parent')`             |
| `:has(selector)`  | 包含特定選擇器的元素        | `$('div:has(p)')`            |
| `:hidden`         | 隱藏的元素                  | `$('div:hidden')`            |
| `:visible`        | 可見的元素                  | `$('div:visible')`           |

### 表單選擇器
| 選擇器              | 說明                       | 示例                         |
|-------------------|----------------------------|------------------------------|
| `:input`          | 所有輸入元素 (包括input、textarea、select、button)                | `$(':input')` 跟 `$('input')` 不同               |
| `:text`           | 文本輸入框                  | `$('input:text')`            |
| `:password`       | 密碼輸入框                  | `$('input:password')`        |
| `:radio`          | 單選按鈕                    | `$('input:radio')`           |
| `:checkbox`       | 複選框                      | `$('input:checkbox')`        |
| `:submit`         | 提交按鈕                    | `$('input:submit')`          |
| `:reset`          | 重置按鈕                    | `$('input:reset')`           |
| `:button`         | 按鈕                        | `$(':button')`               |
| `:file`           | 文件上傳框                  | `$('input:file')`            |
| `:enabled`        | 啟用的表單元素              | `$('input:enabled')`         |
| `:disabled`       | 禁用的表單元素              | `$('input:disabled')`        |
| `:checked`        | 選中的表單元素              | `$('input:checked')`         |
| `:selected`       | 選中的選項元素              | `$('option:selected')`       |

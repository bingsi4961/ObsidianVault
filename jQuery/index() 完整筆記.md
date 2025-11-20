---
title: index() 完整筆記
tags: [jQuery]

---

# index() 完整筆記

## 基本概念

jQuery 的 `index()` 方法用於查找 `指定元素` 在 `相關元素集合` 中的位置索引（從 0 開始計數）。此方法在處理 DOM 元素集合時非常實用，可快速確定元素在結構中的相對位置。

## 三種使用語法

### 1. 無參數用法
```javascript
$(selector).index()
```
返回：所選元素在其同層級兄弟元素中的索引位置。

### 2. 選擇器字符串參數
```javascript
$(selector).index("選擇器字符串")
```
返回：`所選元素` 在所有匹配「選擇器字符串」的 `元素集合` 中的索引位置。

### 3. DOM 元素或 jQuery 對象參數
```javascript
$(selector).index(DOM元素或jQuery對象)
```
返回：指定的 `DOM元素` 或 `jQuery對象` 在所選元素集合中的索引位置。

## 實例展示

### 例子 1：無參數用法
```javascript
// HTML: <ul><li>項目一</li><li>項目二</li><li id="test">項目三</li><li>項目四</li></ul>
var index = $("#test").index(); // 返回 2
```
說明：查找 id="test" 的元素在其同層兄弟元素中的位置。

### 例子 2：選擇器字符串參數
```javascript
// HTML: 
// <div>第一個 div</div>
// <div class="container">第二個 div</div>
// <div>第三個 div</div>
var index = $(".container").index("div"); // 返回 1
```
說明：在所有 div 元素集合中查找 class="container" 的元素的位置。

### 例子 3：DOM 元素或 jQuery 對象參數
```javascript
// HTML: <div><p>第一段</p><p id="target">第二段</p><p>第三段</p></div>
var targetElement = document.getElementById("target");
var index = $("p").index(targetElement); // 返回 1
```
說明：在所有 p 元素的集合中查找 id="target" 的元素的位置。

### 例子 4：混合層級元素
```html
<ul>
    <li>項目 1</li>
    <li>項目 2
        <ul>
            <li class="nested">巢狀項目 1</li>
            <li>巢狀項目 2</li>
        </ul>
    </li>
    <li id="targetItem">項目 3</li>
</ul>
```

```javascript
var index = $("#targetItem").index("li"); // 返回 4
```
說明：在 DOM 中的所有 li 元素集合中（按 DOM 遍歷順序排列）查找 id="targetItem" 的元素位置。

### 例子 5：查找元素在其同級中的位置
```html
<div id="container">
    <p>第一段</p>
    <span>一些文字</span>
    <p id="target">第二段</p>
    <div>一個區塊</div>
</div>
```

```javascript
var index = $("#target").parent().children().index($("#target")); // 返回 2
```
說明：
1. `$("#target").parent()` 選取父元素
2. `.children()` 選取所有子元素
3. `.index($("#target"))` 查找 jQuery 對象 $("#target") 在這個子元素集合中的位置

## 參數類型重要區別

1. **無參數**：僅查找元素在同層兄弟中的位置
2. **字符串參數**：在匹配字符串選擇器的所有元素中查找所選元素位置
3. **DOM/jQuery 對象參數**：在所選元素集合中查找特定 DOM/jQuery 對象的位置

## 注意事項

1. 索引始終從 0 開始計數
2. 找不到元素時返回 -1
3. ==不同參數形式產生截然不同的行為結果==
4. 處理複雜 DOM 結構時，會按 DOM 先序遍歷順序建立扁平化集合
5. 選擇適合需求的參數形式：
   - 無參數：僅查找同級位置
   - 字符串選擇器：在頁面全域中定位
   - DOM/jQuery 對象：在特定集合中定位
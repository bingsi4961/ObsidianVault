---
title: siblings() 方法筆記 / 其他找尋元素的方法
tags: [jQuery]

---


## 一、siblings() 方法基本介紹

`.siblings()` 方法用於選取與當前選定元素相同層級的所有兄弟元素（有相同父元素的其他元素）。

### 基本語法
```javascript
$(selector).siblings([filter])
```

- `selector`：選取的目標元素
- `filter`：可選參數，用於進一步篩選兄弟元素

### 常見用法

#### 1. 選取所有兄弟元素
```javascript
// 選取所有 li 元素的兄弟元素
$("li.active").siblings();
```

#### 2. 帶過濾器的用法
```javascript
// 只選取 class 為 "important" 的兄弟元素
$("li.active").siblings(".important");

// 選取所有 p 標籤的兄弟元素
$("#targetElement").siblings("p");
```

#### 3. 實際應用案例

##### 切換選單的活動狀態
```javascript
// 點擊選單項目時，為其添加 active 類別並從兄弟元素移除
$("#menu li").click(function() {
    $(this).addClass("active");
    $(this).siblings().removeClass("active");
});
```

## 二、jQuery 選擇器行為說明

### 後代選擇器
`$("#menu li")` 會選取 `#menu` 元素下的**所有** `li` 元素，包括：
- 直接子元素
- 孫元素
- 所有更深層的後代元素

#### 範例 HTML：
```html
<ul id="menu">
    <li>項目 1</li>
    <li>項目 2
        <ul>
            <li>子項目 2-1</li>
            <li>子項目 2-2</li>
        </ul>
    </li>
    <li>項目 3</li>
</ul>
```

#### 選擇器行為：
- `$("#menu li")` 會選取所有 5 個 `li` 元素：
  - 「項目 1」
  - 「項目 2」
  - 「子項目 2-1」
  - 「子項目 2-2」
  - 「項目 3」

### 子元素選擇器
`$("#menu > li")` 只會選取 `#menu` 元素的直接子元素，不包括更深層的後代元素。

#### 選擇器行為：
- `$("#menu > li")` 只會選取 3 個直接子元素：
  - 「項目 1」
  - 「項目 2」
  - 「項目 3」

## 三、siblings() 方法與選擇器的關係

`.siblings()` 只會選取與當前元素**同一層級**的兄弟元素：

```javascript
// 如果選取「子項目 2-1」並使用 siblings()
$("子項目 2-1的選擇器").siblings();
```

這只會選取「子項目 2-2」，因為它們是同一層級的兄弟元素。

## 四、其他相關選擇器方法比較

| 方法 | 作用 |
|------|------|
| `.siblings()` | 選取所有兄弟元素 |
| `.next()` | 選取下一個兄弟元素 (只考慮直接相鄰的單個兄弟元素) |
| `.prev()` | 選取前一個兄弟元素 (只考慮直接相鄰的單個兄弟元素) |
| `.prevAll()` | 選取當前元素之前的所有同層級兄弟元素 |
| `.nextAll()` | 選取當前元素之後的所有同層級兄弟元素 |
| `.parent()` | 選取直接父元素（僅一層） |
| `.parents([selector])` | 選取所有祖先元素（多層），可選擇性地用選擇器篩選 |
| `.closest(selector)` | 當前元素開始向上遍歷DOM樹，選取符合選擇器的最近(非第一個)的祖先元素（包含自身） |
| `.children()` | 選取直接子元素（不包括孫元素） |
| `.find()` | 選取符合條件的所有後代元素 |

## 五、元素選取的最佳實踐

### 1. 使用更具體的選擇器
```javascript
// 子元素選擇器，只選取直接子元素
$("#menu > li");
```

### 2. 使用 .children() 方法
```javascript
// 只選取直接子元素
$("#menu").children("li");
```

### 3. 使用 .find() 方法
```javascript
// 選取所有後代元素
$("#menu").find("li");
```

### 4. 效能考量
在大型 DOM 結構中，為提高效能可以快取選擇器結果：

```javascript
// 快取結果以提高效能
var $menuItems = $("#menu > li"); // 注意使用子元素選擇器更精確

$menuItems.click(function() {
    var $this = $(this);
    $this.addClass("active");
    $menuItems.not($this).removeClass("active");
});
```

## 六、實際完整範例

```javascript
<!DOCTYPE html>
<html>
<head>
    <title>jQuery 選擇器與 siblings() 示範</title>
    <style>
        .active { background-color: #ffd700; }
        .menu-item { padding: 10px; margin: 5px; cursor: pointer; }
    </style>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
    <ul id="menu">
        <li class="menu-item">項目 1</li>
        <li class="menu-item">項目 2
            <ul>
                <li class="menu-item">子項目 2-1</li>
                <li class="menu-item">子項目 2-2</li>
            </ul>
        </li>
        <li class="menu-item">項目 3</li>
    </ul>

    <script>
        $(document).ready(function() {
            // 只選取頂層選單項目
            $("#menu > li.menu-item").click(function() {
                $(this).addClass("active");
                $(this).siblings().removeClass("active");
            });
            
            // 選取第二層選單項目
            $("#menu > li > ul > li.menu-item").click(function(e) {
                e.stopPropagation(); // 阻止事件冒泡到父元素
                $(this).addClass("active");
                $(this).siblings().removeClass("active");
            });
        });
    </script>
</body>
</html>
```
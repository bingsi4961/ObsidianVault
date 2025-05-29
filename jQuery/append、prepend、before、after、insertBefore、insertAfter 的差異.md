---
title: append、prepend、before、after、insertBefore、insertAfter 的差異
tags: [jQuery]

---

讓我解釋這些 jQuery 方法的主要差異：

```html
<div id="target">
  <p>現有內容</p>
</div>
```

1. append/appendTo
```javascript
// 在目標元素內部的最後面插入
$("#target").append("<span>新內容</span>");
// 等同於
$("<span>新內容</span>").appendTo("#target");

結果:
<div id="target">
  <p>現有內容</p>
  <span>新內容</span>
</div>
```

2. prepend/prependTo
```javascript
// 在目標元素內部的最前面插入
$("#target").prepend("<span>新內容</span>");
// 等同於
$("<span>新內容</span>").prependTo("#target");

結果:
<div id="target">
  <span>新內容</span>
  <p>現有內容</p>
</div>
```

3. before/insertBefore
```javascript
// 在目標元素之前插入
$("#target").before("<div>新內容</div>");
// 等同於
$("<div>新內容</div>").insertBefore("#target");

結果:
<div>新內容</div>
<div id="target">
  <p>現有內容</p>
</div>
```

4. after/insertAfter
```javascript
// 在目標元素之後插入
$("#target").after("<div>新內容</div>");
// 等同於
$("<div>新內容</div>").insertAfter("#target");

結果:
<div id="target">
  <p>現有內容</p>
</div>
<div>新內容</div>
```

主要區別：
- append/prepend：在元素內部插入
- before/after：在元素外部插入
- 方法名稱相反（如 append/appendTo）：只是插入對象和目標的順序相反，結果相同
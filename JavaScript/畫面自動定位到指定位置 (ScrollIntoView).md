---
title: 畫面自動定位到指定位置 (ScrollIntoView)
tags: [JavaScript]

---

```htmlmixed
<tr scrollElementId="@ftt.Id">
    <td>xxx</td>
</tr>
```
```javascript
// $(function(){ });
$(document).ready(function(){
    const urlParams = new URLSearchParams(window.location.href);
    var scrollElementId = urlParams.get('scrollElementId');        
    if (scrollElementId) {
        var $scrollElement = $(`tr[scrollElementId=${scrollElementId}]`);
        $scrollElement.get(0).scrollIntoView({behavior: 'smooth', block: 'center'});
    }
});
```
`scrollIntoView()` 方法的選項參數中，`behavior` 和 `block` 屬性用於控制滾動的方式和目標元素在視窗中的位置：

### `behavior` 選項

這個屬性控制滾動的動畫效果：

- `'auto'`：默認值，立即滾動到指定位置，沒有平滑過渡效果
- `'smooth'`：平滑滾動到指定位置，有動畫過渡效果

### `block` 選項

這個屬性控制元素在垂直方向上的對齊方式：

- `'start'`：將元素對齊到視窗的頂部
- `'center'`：將元素居中於視窗
- `'end'`：將元素對齊到視窗的底部
- `'nearest'`：默認值，如果元素已經在視窗內，則不滾動；如果不在視窗內，則滾動到最近的位置（頂部或底部）

### 還有一個沒提到的 `inline` 選項

控制元素在水平方向上的對齊方式：

- `'start'`：將元素對齊到視窗的左側
- `'center'`：將元素水平居中於視窗
- `'end'`：將元素對齊到視窗的右側
- `'nearest'`：默認值，同上述 `block` 的 `'nearest'`

### 使用示例

```javascript
// 平滑滾動並將元素居中顯示
element.scrollIntoView({behavior: 'smooth', block: 'center'});

// 平滑滾動並將元素對齊到視窗頂部
element.scrollIntoView({behavior: 'smooth', block: 'start'});

// 平滑滾動並將元素對齊到視窗底部
element.scrollIntoView({behavior: 'smooth', block: 'end'});

// 立即滾動並將元素水平和垂直居中
element.scrollIntoView({behavior: 'auto', block: 'center', inline: 'center'});
```

瀏覽器兼容性說明：`behavior: 'smooth'` 在較老的瀏覽器可能不支持，如果需要支持這些瀏覽器，可能需要使用 polyfill 或第三方滾動庫。
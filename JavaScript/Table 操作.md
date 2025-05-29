---
title: Table 操作
tags: [JavaScript]

---

# parentNode、cellIndex
```htmlembedded=
<table border="1">
  <tr>
    <td>Cell 1</td> <!-- cellIndex = 0 -->
    <td colspan="2">Cell 2+3</td> <!-- cellIndex = 1 -->
    <td>Cell 4</td> <!-- cellIndex = 2 -->
  </tr>
  <tr>
    <td>A</td> <!-- cellIndex = 0 -->
    <td>B</td> <!-- cellIndex = 1 -->
    <td>C</td> <!-- cellIndex = 2 -->
    <td>D</td> <!-- cellIndex = 3 -->
  </tr>
</table>
```

```javascript=
$("td").on("click", function () {
  let cell = this;  // this 是DOM元素
  let row = cell.parentNode;

  alert(cell.cellIndex);
  alert(row.cells[cell.cellIndex].colSpan);
  // alert(cell.colSpan);
});

// 也可以用 get() 方法取得 DOM 元素後再使用
// alert($("td").get(0).cellIndex);
// alert($("td")[0].cellIndex);
```
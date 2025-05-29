---
title: 拉動Scrollbar時，固定行數與欄位
tags: [CSS]

---

```htmlmixed=
<style type="text/css">
    
    tr {                    
        border-top: 1px solid #ccd1d9;
        background-color: #ffffff;  /* 基礎背景色 */
    }

    tr:hover {
        background-color: #f5f5f5;
    }

    th {
        text-align: right;
        min-width: 150px !important;
        padding-right: 10px !important;
    }

    td {
        min-width: 250px !important;
    }

    th.RightBorder {
        border-right: 1px solid #ccd1d9;
    }

    /* 第一欄固定 */
    th.RightBorder {
        position: sticky;
        left: 0;
        background-color: inherit; /* 繼承TR的背景色 */
        z-index: 1;
        border-right: none;
    }

    /* 創建固定的右邊框 */
    /* 為了在拉動 Scrollbar 時，可以呈現右邊線條 */
    th.RightBorder::after {
        content: '';
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        width: 1px; /* 邊框寬度 */
        background-color: #ccd1d9; /* 邊框顏色，根據需要調整 */
        z-index: 1;
    }
    
    /* 為每一個固定行設定位置 */
    tr.fixed-section-1 {
        position: sticky;
        top: 20px;
        background-color: #6ca6d6;
        z-index: 2;
    }

    tr.fixed-section-2 {
        position: sticky;
        top: 71px; /* 根據第一行的高度調整 */
        background-color: #cceef7;
        z-index: 2;
    }

    /* 固定行和列交叉處 */
    tr.fixed-section-1 th.RightBorder,
    tr.fixed-section-2 th.RightBorder {
        z-index: 3;
    }
    
</style>

<table>
    <tbody>
        <tr>
            <th class="RightBorder"></th>
            <td></td>
            <td></td>
            <td></td>			
        </tr>
        <tr class="fixed-section-1">
            <th class="RightBorder"></th>
            <td></td>
            <td></td>
            <td></td>			
        </tr>
        <tr class="fixed-section-2">
            <th class="RightBorder"></th>
            <td></td>
            <td></td>
            <td></td>			
        </tr>
        <tr>
            <th class="RightBorder"></th>
            <td></td>
            <td></td>
            <td></td>			
        </tr>
        <tr>
            <th class="RightBorder"></th>
            <td></td>
            <td></td>
            <td></td>			
        </tr>
    </tbody>
</table>
```
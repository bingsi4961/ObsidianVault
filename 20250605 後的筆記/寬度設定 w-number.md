---
date: 2025-06-15 23:23
aliases: 
tags:
  - BootStrap
  - BootStrap_4_3_1
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
## 可用的 w-* 類別

- `w-25` - 寬度 25%
- `w-50` - 寬度 50%
- `w-75` - 寬度 75%
- `w-100` - 寬度 100%
- `w-auto` - 自動寬度

## 使用範例

```html
<!-- 寬度 25% -->
<div class="w-25 bg-primary">25% 寬度</div>

<!-- 寬度 50% -->
<div class="w-50 bg-secondary">50% 寬度</div>

<!-- 寬度 75% -->
<div class="w-75 bg-success">75% 寬度</div>

<!-- 寬度 100% -->
<div class="w-100 bg-info">100% 寬度</div>

<!-- 自動寬度 -->
<div class="w-auto bg-warning">自動寬度</div>
```

## 在 .NET MVC 中的應用

在你的 Razor 視圖中可以這樣使用：

```html
@{
    ViewData["Title"] = "Bootstrap 寬度範例";
}

<div class="container">
    <div class="row">
        <div class="col-12">
            <h2>產品清單</h2>
            <!-- 表格佔滿整個容器寬度 -->
            <table class="table w-100">
                <thead>
                    <tr>
                        <th>產品名稱</th>
                        <th>價格</th>
                    </tr>
                </thead>
                <tbody>
                    @foreach(var product in Model.Products)
                    {
                        <tr>
                            <td>@product.Name</td>
                            <td>@product.Price</td>
                        </tr>
                    }
                </tbody>
            </table>
            
            <!-- 按鈕組，各佔不同寬度 -->
            <div class="mb-2">
                <button class="btn btn-primary w-25">新增</button>
                <button class="btn btn-secondary w-auto">編輯</button>
            </div>
        </div>
    </div>
</div>
```

## 搭配 JavaScript/jQuery 使用

```javascript
$(document).ready(function() {
    // 動態調整元素寬度
    $('#adjustWidth').click(function() {
        $('#targetDiv').removeClass('w-50').addClass('w-100');
    });
    
    // 根據視窗大小調整寬度
    $(window).resize(function() {
        if ($(window).width() < 768) {
            $('.responsive-width').removeClass('w-50').addClass('w-100');
        } else {
            $('.responsive-width').removeClass('w-100').addClass('w-50');
        }
    });
});
```


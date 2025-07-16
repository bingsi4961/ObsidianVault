---
title: Html Helper、Url Helper、Ajax Helper
tags: [ASP.NET MVC 5]

---

# @Html.ActionLink // 創建超連結
```csharp
@Html.ActionLink("刪除文字", "Delete", "SpmCostKpCols",
    new {
        id = item.Id,
        Model.QueryCategory,
        Model.PageIndex,
        Model.SortCol,
        Model.SortBy
    }, 
    new {
        @class = "btn btn-primary",
        onclick = "return confirm('確認刪除?');"
    }
)
```

```htmlmixed=
<a class="btn btn-primary" 
   href="/SpmCostKpCols/Delete/9?QueryCategory=HDD&amp;PageIndex=1&amp;SortCol=ModifyDate&amp;SortBy=desc" 
   onclick="return confirm('確認刪除?');">
   刪除文字
</a>
```

# @Url.Action // 生成 URL 字串
```csharp
@Url.Action("Search", "Product", 
    new { 
        id = 18,
        category = "Electronics",
        price = 100,
        sort = "desc" 
    }
)
```

```html
/Product/Search/18?category=Electronics&price=100&sort=desc
```

# @Url.Content() // 產生靜態資源的路徑（如圖片、CSS、JavaScript 等）
```csharp=
// ~ 代表網站根目錄

// CSS 檔案
<link href="@Url.Content("~/Content/css/site.css")" rel="stylesheet" />

// JavaScript 檔案
<script src="@Url.Content("~/Scripts/jquery.js")"></script>

// 圖片
<img src="@Url.Content("~/Images/logo.png")" alt="Logo" />

// 整合 asp-append-version
<script src='@Url.Content("~/js/TableSort.js")' asp-append-version="true"></script>    
// <script src="/js/TableSort.js?v=b1IjxpaPHALYugiW21zrwF6N6l_GnfWp6nvH6cfSbfQ"></script>    
```
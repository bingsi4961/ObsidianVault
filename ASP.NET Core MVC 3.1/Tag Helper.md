---
title: Tag Helper
tags: [ASP.NET Core MVC 3.1]

---

```htmlembedded=
<a asp-controller="Home" asp-action="Index">首頁</a>
<a href="/Home/Index">首頁</a> 

<a asp-controller="Product" asp-action="Details" asp-route-id="5">查看產品詳細</a>
<a href="/Product/Details/5">查看產品詳細</a>

<a asp-area="Admin" asp-controller="Dashboard" asp-action="Index">管理面板</a>
<a href="/Admin/Dashboard/Index">管理面板</a> 

<link href="~/css/site.css" asp-append-version="true" />
<link href="/css/site.css?v=STjaWhxFesmmKykdt9sVwOwGz81-Jf1V7aqflufWQUc" rel="stylesheet" />
若css檔有變動，v的值(版本值)也會變化，以利client重新自Server抓取，不Cache
asp-append-version 會根據檔案內容產生雜湊值

```
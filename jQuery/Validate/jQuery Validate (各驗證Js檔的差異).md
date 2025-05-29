---
title: jQuery Validate (各驗證Js檔的差異)
tags: [jQuery]

---

1. `jquery.validate.min.js`
   - 這是 jQuery Validate 插件的核心文件
   - 包含基本的驗證功能，如：required、minlength、maxlength、email、number 等
   - 提供了最基本且常用的表單驗證方法
   - 這個是必需的基礎文件

2. `additional-methods.min.js`
   - 這是 jQuery Validate 插件的擴展驗證方法集
   - 提供額外的驗證方法，例如：
     * 信用卡驗證
     * 日期格式驗證
     * 文件擴展名驗證
     * 更多複雜的正則表達式驗證
   - 是可選的，只有當需要使用這些額外的驗證方法時才需要引入

3. `jquery.validate.unobtrusive.min.js`
   - 這是微軟為 ASP.NET MVC 開發的一個擴展驗證插件
   - 主要用於將 MVC 的服務端驗證規則自動轉換為客戶端驗證
   - 允許通過 data attributes 來定義驗證規則，使驗證代碼更加整潔
   - 特別適合在 ASP.NET MVC 專案中使用
   - 如果不是使用 ASP.NET MVC，通常不需要引入這個文件

使用建議：
- 如果是一般的 Web 專案，只需要基本驗證功能，使用 `jquery.validate.min.js` 就足夠
- 如果需要更多進階的驗證方法，可以加入 `additional-methods.min.js`
- 如果是 ASP.NET MVC 專案，則建議使用 `jquery.validate.unobtrusive.min.js`
---
date: 2026-05-25 07:53
title:
aliases:
tags:
  - Validation
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[]]

---

| 檔案                                   | 說明                                                                                     | 必要性            |
| ------------------------------------ | -------------------------------------------------------------------------------------- | -------------- |
| `jquery.validate.min.js`             | 核心驗證功能，提供 required、email、number、minlength、maxlength 等基本規則                              | 必要             |
| `additional-methods.min.js`          | 擴充驗證方法，提供信用卡、日期格式、副檔名等進階規則                                                             | 選用             |
| `jquery.validate.unobtrusive.min.js` | 微軟為 ASP.NET MVC 開發的橋接插件，將後端驗證規則透過 `data-val-*` 屬性自動轉換為前端驗證。**非 ASP.NET MVC 專案通常不需要引入** | ASP.NET MVC 專用 |

### 使用建議

- **一般 Web 專案**：只需引入 `jquery.validate.min.js`
- **需要進階驗證**：再加入 `additional-methods.min.js`
- **ASP.NET MVC 專案**：三個檔案依序引入

```html
<script src="jquery.validate.min.js"></script>
<script src="jquery.validate.unobtrusive.min.js"></script>
<!-- 若需要進階驗證規則，再加入 -->
<script src="additional-methods.min.js"></script>
```
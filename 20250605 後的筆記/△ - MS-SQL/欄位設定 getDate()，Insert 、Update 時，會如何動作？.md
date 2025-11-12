---
date: 2025-11-07 18:34
aliases:
tags:
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

在 MSSQL 中，如果欄位設定了預設值為 GETDATE()：

1. INSERT 時：
   - 如果沒有指定該欄位的值，會自動填入當下的日期時間
   - 如果有指定值，則會使用指定的值

2. UPDATE 時：
   - 預設值只會在 INSERT 時生效
   - UPDATE 時不會自動更新，除非你明確在 UPDATE 語句中設定該欄位的值
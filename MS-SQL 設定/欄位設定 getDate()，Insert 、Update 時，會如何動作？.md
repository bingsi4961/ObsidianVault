---
title: 欄位設定 getDate()，Insert 、Update 時，會如何動作？
tags: [MS-SQL 設定, 技術文件]

---

在 MSSQL 中，如果欄位設定了預設值為 GETDATE()：

1. INSERT 時：
   - 如果沒有指定該欄位的值，會自動填入當下的日期時間
   - 如果有指定值，則會使用指定的值

2. UPDATE 時：
   - 預設值只會在 INSERT 時生效
   - UPDATE 時不會自動更新，除非你明確在 UPDATE 語句中設定該欄位的值
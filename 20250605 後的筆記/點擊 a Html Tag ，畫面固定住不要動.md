---
date: 2025-07-04 11:27
aliases: 
tags:
  - HTML
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

如果點擊這個連結，畫面位置會跳到畫面最上方
```html
<a href="#" onclick="deleteBaseline(10);">Delete</a>
```

如果點擊這個連結，畫面位置不會動
```html
<a href="javascript:void(0)" onclick="deleteBaseline(10);">Delete</a>
```
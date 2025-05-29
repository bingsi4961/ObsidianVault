---
title: Java SE6 技術手冊
tags: [入庫]

---

# Java SE6 技術手冊
###### tags: `書籍筆記`

---
## P.7-19
> 類別成員(class)
>> 1. 資料成員(Field)
>> 1. 方法成員(Method)
>> 1. 靜態資料成員 <= 多個物件的共享資料 / Math.PI、Integer.MAX_VALUE
>> 1. 靜態方法成員 <= 共享的工具方法(類別可以解釋為工具類別) / Math.Exp()、Math.Log()、Math.Sin()、Integer.parseInt()

:::info
1.　方法成員中，有隱含this參考名稱。
2.　靜態方法成員中，==沒有==隱含this參考名稱。
:::
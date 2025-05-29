---
title: 圖解LINUX指令操作與網路設定(電子書)
tags: [書籍筆記]

---

# 圖解LINUX指令操作與網路設定(電子書)
###### tags: `書籍筆記`



- [CentOS 7 修改語系與鍵盤配置 ](https://blog.twtnn.com/2015/10/centos-7.html)
- [linux下.swp文件是什麼？](https://www.twblogs.net/a/5c2212c4bd9eee16b3daefa1)


---
## 第1章 在開始學習之前

```javascript=
Unix -> BSD、macOS雛型、Linux
Linux -> Debian Ubuntu、Red Hat CentOS
```

```javascript=
Unix -> macOS -> iOS (Apple)
Unix -> Linux -> 【Red Hat Enterprise Linux】、【Red Hat Fedora】、【Red Hat CentOS】
Unix -> Linux -> 【Debian DebianGNU/Liunx】、【Debian Ubuntu】
Unix -> OpenBSD
Unix -> Solaris (Oracle)
Linux -> Android (Google)
```

```javascript=
-- 主要的付費版本 --
Red Hat Enterprise Linux (支援10年)
SUSE Linux Enterprise Server
```

```javascript=
-- 主要的免費版本 --
Red Hat CentOS (支援10年)
```

```javascript=
Debian Ubuntu (一般) 支援9個月
Debian Ubuntu (LTS) 支援5年
Red Hat Fedora 支援13個月
```
---
## 第2章 開始使用Linux吧
```bash=
[rinako@localhost ~]$date
# [username@host 目前目錄]$指令
```

```bash=
# 指令
$date       # 顯示今天的日期
$cal        # 顯示本月的月曆
$cal 2020   # 顯示2020年的月曆
$cal 4 2020 # 顯示2020年4月的月曆
$cal -m     # 一週的開頭從星期日換成星期一
$cal -3 6 2023    # 以2020年的6月為中間月份，顯示三個月的月曆
$man cal    # 查詢 cal的使用方法
$cal --help # 顯示參數使用方式的簡略說明
$exit       # 登出Linux (不代表關機)
```

---
## 第3章 檔案與目錄的基本操作

```
1. ~ 代表個人目錄 (/home/user)
2. (.點符號) 代表目前目錄
3. 檔名規則允許字元：[A-Z][a-z][0-9][_][-][.] / 大小寫視為不同(包括副檔名)
4. Linux沒有一定要加上副檔名，但建議加上(txt 文字檔、conf 設定檔、bak 備份檔、[沒有副檔名]目錄)
```


| 目錄名稱 | 說明                 |
|----------|----------------------|
| /        | 根目錄               |
| bin      | 一般使用者可用的命令 |
| sbin     | 管理者專用的命令     |
| home     | 一般使用者專用的目錄 |
| root     | 管理者的個人目錄     |
| etc      | Linux系統相關的檔案  |
| sys      | 系統相關目錄         |
| boot     | 啟動專用檔案         |
| tmp      | 暫存專用目錄         |
| var      | 歷程檔案             |


---
### man [想查詢的指令]：說明手冊　
:::info
【查詢結果畫面操作】
- <kbd>Space鍵</kbd> 到下一頁
- <kbd>b鍵</kbd> 回上一頁
- <kbd>q鍵</kbd> 關閉說明畫面
:::

---
### [想查詢的指令] --help：簡略說明

---
### pwd：顯示目前的工作目錄

---
### cd [Path]：切換路徑
| 指令範例   | 說明          |
|----------|-------------- |
| cd ..    | 回到上一層目錄  |
| cd ../.. | 回到上兩層目錄  |
| cd ~     | 回到個人目錄    |
| cd       | 回到個人目錄    |

---
### ls：顯示檔案及目錄 (以英文字母排序A-Z)
:::info
* -l：顯示檔案或目錄的進階資訊
* -r：倒序顯示
* -F：讓檔案還是目錄更容易辦識 (目錄名稱會加上/斜線符號)
* -d：顯示目錄本身名稱，不會顯示目錄的內容 (沒什麼用處)
* -dl：承上，顯示目錄的進階資訊
* -t：依照修改時間排序(由近到遠)
* -R：顯示指定目錄及 "子目錄"的檔案 (R是遞迴概念)
* -a：顯示隱藏檔案 (通常以.(點)為字首 => .bashrc)
:::

```bash=
## 指令範例
ls        依英文字母 "正序" 顯示
ls -r     依英文字母 "倒序" 顯示
ls -t     依修改時間 "倒序" 顯示
ls -tr    依修改時間 "正序" 顯示
ls -F     讓檔案還是目錄更容易辦識，目錄名稱會加上/斜線符號
ls ~ -lF  在個人目錄中，顯示檔案及目錄的進階資訊，並且目錄名稱加上/斜線符號
ls ~ -dl  顯示個人目錄的進階資訊
ls -l --time-style=+%c    顯示更進階的時間資訊
ls -a     顯示隱藏檔案及目錄(通常以.(點)為字首 => .bashrc)
```

```bash=
## 建議習慣
ls [Folder] -Fl   依英文字母 "正序" 顯示 ★
ls [Folder] -Flr  依英文字母 "倒序" 顯示
ls [Folder] -Flt  依修改時間 "倒序" 顯示 ★
ls [Folder] -Fltr 依修改時間 "正序" 顯示
```

---
### touch：新增檔案

---
### cat：顯示檔案內容(內容不長)
:::info
* -n：加上行號
:::

---
### less：顯示檔案內容(內容較長)
:::info
【查詢結果畫面操作】
- <kbd>F鍵</kbd>、<kbd>Space鍵</kbd> 到下一頁
- <kbd>B鍵</kbd> 回上一頁
- <kbd>Up鍵</kbd> 回上一行
- <kbd>Down鍵</kbd> 往下一行
- <kbd>Q鍵</kbd> 關閉說明畫面
:::

---
### cp [SrcFile] [DestFile]：複製　　　　
:::info
* -v：顯示複製結果
* -i：目的地有相同檔名，則跳出警示訊息 (回覆 y/n)
* -r：包括子目錄(遞迴概念)
:::

| **指令範例**                         | **說明**                                               |
|----------------------------------|-------------------------------------------------------|
| cp [nikki.txt] [doc/]            | 複製檔案到 doc/ 下                                          |
| cp [nikki.txt] [doc/nikki2.txt]  | 複製並更名 (若目的地有相同檔名，則直接覆寫)                               |
| cp [a.txt b.txt c.txt] [doc2/]   | doc2/ 必需存在，同時複製多個檔案                                   |
| cp -r [a.txt doc/ b.txt] [doc2/] | doc2/ 必需存在，同時複製多個檔案及目錄 (需要加上參數 -r)                    |
| cp -r [doc/*] [doc2/]            | doc2/ 必需存在，將 doc/ 下所有檔案、子目錄，複製到 doc2 /下 => doc2/a.txt |
| cp -r [doc/] [doc2/]             | ★ 若 doc2/ 已存在，將 doc/ 整個複製到 doc2/ 下 => doc2/doc/a.txt  |
| cp -r [doc/] [doc2/]             | ★ 若 doc2/ 不存在，將 doc/ 整個複製，並更名為 doc2/ => doc2/a.txt    |


```bash=
## 建議習慣
cp -vir [SrcFile] [DestFile]
```

---
### mv [Src] [Dest]：移動檔案、變更檔名
:::info
* -v：顯示複製結果
* -i：目的地有相同檔名，則跳出警示訊息 (回覆 y/n)
:::

```bash=
## 指令範例
# 目錄名稱後面的 / 符號，可加可不加
mv -vi [nikki2.txt] [nikki3.txt] 變更檔案名稱 && 避免覆寫相同檔名 && 顯示執行結果
mv [nikki2.txt] [doc/] 將檔案搬移至目錄下

# doc2/若存在，則將 doc1/ 移動至 doc2/下
# doc2/若不存在，則將 doc1/ 更名為 doc2/
# 不需要[-r]參數來包括子目錄
mv [doc1/] [doc2/] 變更目錄名稱 
```

```bash=
## 建議習慣
mv -vi [Src] [Dest]
```

---
### mkdir：新建目錄

---
### rmdir：移除目錄 (目錄必須為空)

---
### rm -r：移除目錄 (可以刪除還有檔案的目錄及子目錄)

---
## 第4章 (vi) 第一次使用編輯器就上手

```bash=
$vi    # 啟動vi編輯器
$vim   # 啟動VIM編輯器
$vi ~/lincoln.txt   # 在vi編輯器開啟檔案
```
- 從命令模式切換成插入模式 => <kbd>i鍵</kbd>
- 從插入模式切換成命令模式 => <kbd>Esc鍵</kbd>
<br/>

:::success
在插入模式
:::

### 刪除文字
- <kbd>Backspace鍵</kbd> => 刪除游標位置的左側文字
- <kbd>Delete鍵</kbd> => 刪除游標位置的文字

:::info
在命令模式
:::

### 命令
* <kbd>:鍵</kbd> + set number => 顯示列編號
* <kbd>:鍵</kbd> + set nonumber => 不顯示列編號
* <kbd>:鍵</kbd> + set list => 顯示換行符號

### 刪除文字、剪下文字
* <kbd>Shift鍵</kbd> + <kbd>x鍵</kbd> => 刪除游標位置的左側文字
* <kbd>x鍵</kbd> => 刪除 <font color=red>(剪下)</font> 游標位置的文字
* <kbd>3鍵</kbd> , <kbd>x鍵</kbd> => 刪除 <font color=red>(剪下)</font> 游標位置為開始的3個文字
* <kbd>p鍵</kbd> => 在游標位置的<font style= background:#82FF82>右側</font>插入剛才剪下的文字

### 刪除列、剪下列 <font color=blue>(不只到畫面右端，而是到換行的位置)</font>
* <kbd>d鍵</kbd> , <kbd>d鍵</kbd> => 刪除 <font color=red>(剪下)</font> 游標位置的列
* <kbd>5鍵</kbd> , <kbd>d鍵</kbd> , <kbd>d鍵</kbd> => 刪除 <font color=red>(剪下)</font> 游標位置為開始的5個列 <font color=red>(不是刪除剪下5次)</font>
* <kbd>p鍵</kbd> => 在游標位置的<font style= background:#82FF82>下一列</font>貼上剛才複製的列資料

### 複製文字
* <kbd>y鍵</kbd> , <kbd>l鍵</kbd> => 複製游標位置的1個文字
* <kbd>4鍵</kbd> , <kbd>y鍵</kbd> , <kbd>l鍵</kbd> => 複製游標位置為開始的4個文字 <font color=red>(不是複製4次)</font>
* <kbd>p鍵</kbd> => 在游標位置的<font style= background:#82FF82>右側</font>插入剛才複製的文字

### 複製列 <font color=blue>(不只到畫面右端，而是到換行的位置)</font>
* <kbd>y鍵</kbd> , <kbd>y鍵</kbd> => 複製游標位置的列
* <kbd>5鍵</kbd> , <kbd>y鍵</kbd> , <kbd>y鍵</kbd> => 複製游標位置為開始的5個列 <font color=red>(不是複製5次)</font>
* <kbd>p鍵</kbd> => 在游標位置的<font style= background:#82FF82>下一列</font>貼上剛才複製的列

### 貼上
* <kbd>7鍵</kbd> , <kbd>p鍵</kbd> => <font color=red>重複貼上7次</font> 複製的字元或列

### 取消操作
* <kbd>u鍵</kbd> => 取消前一項操作 (undo)
* <kbd>.鍵</kbd> => 重做前一項取消的操作 (redo) / <font color=red>非指取消的動作，而是指取消前的那一個動作</font>

### 儲存檔案
* 儲存新檔案 => <kbd>:鍵</kbd> , <kbd>w鍵</kbd> + <kbd>Space鍵</kbd> + fileName.txt , <kbd>Enter鍵</kbd>
* 儲存舊檔案 => <kbd>:鍵</kbd> , <kbd>w鍵</kbd> , <kbd>Enter鍵</kbd>
* 結束vi編輯器 => <kbd>:鍵</kbd> , <kbd>q鍵</kbd> , <kbd>Enter鍵</kbd>
* 結束vi編輯器而不存檔 => <kbd>:鍵</kbd> , <kbd>q鍵</kbd> , <kbd>!鍵</kbd> , <kbd>Enter鍵</kbd>

### 搜尋
* 從游標位置往<font color=red>後</font>搜尋 => <kbd><font color=red>/</font> 鍵</kbd> + keyword , <kbd>Enter鍵</kbd>
* 跳至<font color=red>下個</font>對應的字串 => <kbd>n鍵</kbd>
* 跳至<font color=red>上個</font>對應的字串 => <kbd>Shift鍵</kbd> + <kbd>n鍵</kbd>
* 從游標位置往<font color=blue>前</font>搜尋 => <kbd><font color=blue>?</font> 鍵</kbd> + keyword , <kbd>Enter鍵</kbd>
* 跳至<font color=blue>上個</font>對應的字串 => <kbd>n鍵</kbd>
* 跳至<font color=blue>下個</font>對應的字串 => <kbd>Shift鍵</kbd> + <kbd>n鍵</kbd>

### 游標移動
* <kbd>H鍵</kbd> 移動至畫面最上方
* <kbd>M鍵</kbd> 移動至畫面最中央
* <kbd>L鍵</kbd> 移動至畫面最下方
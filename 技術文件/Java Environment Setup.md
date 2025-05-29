---
title: Java Environment Setup
tags: [技術文件]

---

# Java Environment Setup
###### tags: `技術文件`

---
## Singles
- [N/A]()

---
## Apache Ant
- [1-ANT-安裝與配置](https://iter01.com/572879.html)
- [Windows下apache ant安裝、環境變數配置教程](https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/320609/)
- [ANT環境安裝設置](http://tw.gitbook.net/ant/ant_environment.html)

---
## Java JDK
- [Java JDK 下載、安裝與環境變數設定教學-Windows 篇](https://www.kjnotes.com/devtools/35)
- [來安裝JDK](https://openhome.cc/Gossip/JavaEssence/InstallJDK.html)
- [Windows 安裝 JDK 開發環境](https://ithelp.ithome.com.tw/articles/10210957)

---
## Eclipse
- [Eclipse汉化 中文语言包下载安装 Babel Language Pack
](https://blog.csdn.net/gzn00417/article/details/104194573)
- [Babel Language Packs Downloads](https://archive.eclipse.org/technology/babel/babel_language_packs/)
- [[教學] 如何讓Eclipse中文化](https://xenby.com/b/99-%E6%95%99%E5%AD%B8-%E5%A6%82%E4%BD%95%E8%AE%93eclipse%E4%B8%AD%E6%96%87%E5%8C%96)
- [After installed 'Babel Language Pack for eclipse in Chinese(Simplified)',the 'Outline' View can't show correctly.](https://bugs.eclipse.org/bugs/show_bug.cgi?id=545652)
- [Eclipse控制台输出中文乱码问题的解决](https://blog.csdn.net/limenghua9112/article/details/50783679)
- [eclipse恢復預設佈局](https://www.796t.com/content/1540964283.html)

// ================================================== //

::: info
==我想在eclipse裡，檢視 apache tomcat api 的程式碼，該如何達成？==

ChatGPT

您可以在 Eclipse 中將 Apache Tomcat 的 API JAR 檔案匯入至 Java Build Path 中，以檢視其程式碼。

以下是匯入 API JAR 檔案的步驟：

1. 下載所需版本的 Apache Tomcat API JAR 檔案（通常在官方網站的下載頁面可以找到）。
1. 在 Eclipse 中開啟專案，並右鍵點擊專案名稱，選擇 Properties。
1. 在 Properties 視窗中，選擇 Java Build Path。
1. 在 Libraries 頁籤中，點擊 Add External JARs 按鈕，並選擇所下載的 Apache Tomcat API JAR 檔案。
1. 在 Libraries 頁籤中，將剛才匯入的 JAR 檔案展開，點擊其源代碼附加資訊（Source Attachment）欄位右側的 Edit 按鈕。
1. 在 Source Attachment Configuration 視窗中，點擊 External File 按鈕，並選擇包含源代碼的壓縮檔（通常是 tar.gz 或 zip 格式）。
1. 點擊 OK 按鈕，完成設定。

完成上述步驟後，您就可以在 Eclipse 中查看 Apache Tomcat API 的程式碼了。您可以在專案中建立一個 Java 檔案，並導入所需的 Tomcat 類別，然後按住 Ctrl 鍵並點擊該類別的方法名稱，即可進入該方法的源代碼檢視頁面。
:::

::: info
==如何知道類別所對應的JAR檔==

在Eclipse編輯中，反白類別名稱，鍵盤Ctrl+Shift+T 打開 Open Type 對話框，從中可得知對應的JAR檔及路徑
:::

// ================================================== //



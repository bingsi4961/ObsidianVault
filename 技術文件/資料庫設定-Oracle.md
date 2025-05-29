---
title: 資料庫設定-Oracle
tags: [技術文件]

---

# 資料庫設定-Oracle
###### tags: `技術文件`

---
## Single
- [SQL Server 與 Oracle 伺服器免費版本資訊整理-黑暗執行緒 (darkthread.net)](https://blog.darkthread.net/blog/sql-oracle-free-editions/)
- [Oracle Database History](https://en.wikipedia.org/wiki/Oracle_Database)
- [Oracle 系統架構基本概念 系統架構基本概念_陳士杰.pdf](https://cko2xt-zoho.quip.com/-/blob/AbaAAAK9w87/70y3mIfYY1fwKilX3lXeIQ?name=Oracle%20%E7%B3%BB%E7%B5%B1%E6%9E%B6%E6%A7%8B%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%20%E7%B3%BB%E7%B5%B1%E6%9E%B6%E6%A7%8B%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5_%E9%99%B3%E5%A3%AB%E6%9D%B0.pdf)
- [Oracle_Install_Uninstall_陳士杰.ppt](https://cko2xt-zoho.quip.com/-/blob/AbaAAAK9w87/xIsW9ch8i1Rve_yFiHGGNA?name=Oracle_Install_Uninstall_%E9%99%B3%E5%A3%AB%E6%9D%B0.ppt)

---
## Download
- [Oracle Database Software Downloads](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html)
- [Oracle SQL Developer Downloads](https://www.oracle.com/database/sqldeveloper/technologies/download/)
- [Oracle Instant Client Downloads for Microsoft Windows (x64) 64-bit](https://www.oracle.com/tw/database/technologies/instant-client/winx64-64-downloads.html)

---
## Oracle DataBase 安裝
- [Oracle Database 11gR2 安裝教學 (適用 Windows) - 理財工程師 Mars](https://blog.hungwin.com.tw/oracle-11gr2-install/)
- [\[DLP\] 安裝與建置 01 安裝和建立 ORACLE 資料庫 | Wellife 創意好物](http://www.wellife.com.tw/blog/2015/07/28/dlp-%E5%AE%89%E8%A3%9D%E8%88%87%E5%BB%BA%E7%BD%AE-01-%E5%AE%89%E8%A3%9D%E5%92%8C%E5%BB%BA%E7%AB%8B-oracle-%E8%B3%87%E6%96%99%E5%BA%AB/)
- [安裝 Oracle Database XE 11gR2](https://www.cnblogs.com/Satu/p/11226782.html)
- [Oracle Database 12C 安裝教程](https://www.uj5u.com/shujuku/4091.html)
- [Oracle 12C安裝 - tw511教學網](https://www.tw511.com/24/257/9297.html)

```java=
Oracle 基本目錄：c:\app
Oracle 軟體位置(Oracle_Home)：{基本目錄}\product\19.0.0\client_1
Oracle DB位置：{基本目錄}\oradata
OCI Library：{Oracle_Home}\bin\oci.dll
sqlnet.ora、tnsname.ora、listener.ora 位置：{Oracle_Home}\network\admin
範例資料庫(*.dbt) 樣板位置：{Oracle_Home}\assistants\dbca\templates
```

```java
Enterprise Manager 資料庫控制 URL
http://user-PC68:1158/em

iSQL*Plus URL
http://user-PC68:5560/isqlplus

iSQL*Plus DBA URL
http://user-PC68:5560/isqlplus/dba
```

```java
網路服務名稱(Net-ServiceName)：tnsnames.ora 裡的組態名稱
服務名稱(ServiceName)：資料庫對外的服務名稱
```

---
### sqlnet.ora
```java=
SQLNET.AUTHENTICATION_SERVICES= (NTS)
NAMES.DIRECTORY_PATH= (TNSNAMES, EZCONNECT, HostName)
```

:::warning
A. NAMES.DIRECTORY_PATH=(EZCONNECT)
: 登入指令必須要有 IP 、PORT、Service_Name
EX:sqlplus account/password@ip(hostname):port/service_name

B. NAMES.DIRECTORY_PATH=(TNSNAMES)
: 本機要搭配 TNSNAMES.ora 設定檔
EX: account/password@net_service_name (tnsnames.ora 組態名稱)

C. NAMES.DIRECTORY_PATH=(HOSTNAME)
: EX: sqlplus account/password@hostname

D. NAMES.DIRECTORY_PATH=(TNSNAMES, EZCONNECT, HostName)
: 當執行指令 sqlplus system/oracle@myoracle，客戶端首先會在 tnsnames.ora 文件中找 myorcle 的組態。如果沒有相應的組態，則往 EZCONNECT 的指令格式去解析. 若發現指令格式錯誤，則往 HOSTNAME 的指令格式去解析，把 myoracle 當作一個主機名稱，通過網路解析它的IP地址並且連線；在這裡還要考慮到Server端是否有設定 GLOBAL_DBNAME=myoracle。
:::




:::info
:book: 筆記
* sqlnet.ora 簡言之，就是選擇如何連線的方式。(TNSNAMES、EZCONNECT、HOSTNAME ....)
:::

---
### tnsname.ora
```clojure=
PRODUCT=(
    (ADDRESS=*)
    (CONNECT_DATA=*)
)

PRODUCT=(
    (ADDRESS_LIST=(ADDRESS=*)(ADDRESS=*)(ADDRESS=*))	
    (CONNECT_DATA=*)
)

ADDRESS=(PROTOCOL=*)(HOST=*)(PORT=*)
CONNECT_DATA=(SERVICE_NAME=*)(SERVER=*)(SID=*)
```

- [Oracle TNS連線設定(Tnsnames.ora)](http://bradctchen.blogspot.com/2019/07/oracle-tnstnsnamesora.html?utm_source=pocket_saves)
- [Oracle tnsname.ora檔的存放位置 via Regedit](https://matthung0807.blogspot.com/2017/09/oracle-tnsnameora.html?utm_source=pocket_saves)
- [Oracle Sql Developer 找不到TNS 設定檔 - 飛天部落](http://dvd333liu.blogspot.com/2015/09/oracle-sql-developer-tns.html)

---
### listener.ora
```csharp=
// LSNR4CKO2：監聽器名稱
// GLOBAL_DBNAM：全域資料庫名稱

SID_LIST_LSNR4CKO2X = (
    SID_LIST = 
        (SID_DESC = (GLOBAL_DBNAME = orcl8.cko2x.tw) (ORACLE_HOME = orclHome1) (SID_NAME = orcl8))
        (SID_DESC = (GLOBAL_DBNAME = orcl9.cko2x.tw) (ORACLE_HOME = orclHome2) (SID_NAME = orcl9))	
)

LSNR4CKO2X = (
    DESCRIPTION_LIST =
        (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = SK0341C195)(PORT = 1521)))
        (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = SK0341C195)(PORT = 1521)))
)
```

---
### sqlnet.ora、tnsnames.ora、listener.ra 的關係
- [Oracle Database 設定檔 listener.ora & sqlnet.ora & tnsnames.ora 三者的關係](https://blog.xuite.net/mistech/blog/211079235-Oracle+Database+%E8%A8%AD%E5%AE%9A%E6%AA%94+listener.ora+%26+sqlnet.ora+%26+tnsnames.ora+%E4%B8%89%E8%80%85%E7%9A%84%E9%97%9C%E4%BF%82?utm_source=pocket_reader#)

:::info
:book: 筆記
* 可透過 oracle Net Configuration Assistant、oracle Net Manager 來編輯 tnsnames.ora、sqlnet.ora、listener.ora
:::


---
## 指令
* connect scott/scott@主機名稱:1521/orcl <= 客戶端不需進行任何配置，直接依主機及ServiceName(SID)進行連線。(NAMES.DIRECTORY_PATH= (EZCONNECT))
* connect scott/scott@orcl(服務命名) <= 客戶端需要配置 tnsnames.ora
* 測試服務能否連得上，用 tnsping <服務命名>。該指令會先使用 sqlnet.ora 參數檔，再從 tnsnames.ora 抓連線字串，進行連線。<服務命名>指的是 tnsnames.ora 中，每個組態的名稱。
* 啟動 SQL*Plus → sqlplus /nolog (先不登入)
* SYS 使用者身分登入 → SQL> connect sys/password@protect as sysdba
---
## 名稱s

:::info
資料庫名稱 (DB_NAME)
::::
1. in parameter file、control files
1. select name from v$database;

:::info
資料庫實例名稱 (instance_name)
::::
1. in parameter file
1. select instance_name from v$instance;
1. 資料庫實例名稱、ORACLE_SID，雖然兩者都表示 Oracle實例，但 instance_name 是 oracle資料庫參數。而 ORACLE_SID 是作業系統的環境變量。
1. 用於和作業系統進行聯繫的標識，也就是說資料庫和作業系統之間的交互用的是資料庫實例名。
1. ORACLD_SID用於與作業系統交互；也就是說，從作業系統的角度訪問實例名，必須通過 ORACLE_SID。
1. OS <--> Service_Name <--> SID (Listener.GLOBAL_DBNAME) <--> Instance <--> Database
1. **且ORACLE_SID必須與instance_name的值一致**

:::info
資料庫網域名稱 (db_domain)
::::
1. in parameter file
1. select value from v$parameter where name = 'db_domain';

::::info
全域資料庫名稱 (Global Database Name)
::::
1. 在網路環境中，用以識別每一個Oracle資料庫的唯一識別名稱(獨一無二)。
1. 建議命名方式 DBName.(DB所在的網域名稱/自行定義名稱) >> orcl.nuu.edu.tw 或 orcl.sjchen
1. orcl => 資料庫名稱、SID
1. nuu.edu.t / sjchen => DB_DOMAIN
1. 系統預設的DBName (資料庫名稱，或稱系統識別名稱SID) 為 orcl
1. 全域資料庫名稱 = 資料庫名稱.資料庫網域名稱

:::info
資料庫服務名稱 (Service_Name)
::::
1. select value from v$parameter where name = 'service_name';
1. 如果資料庫有域名，則資料庫服務名稱就是全域資料庫名稱；否則，資料庫服務名與資料庫名相同。
1. 在cluster環境中，外部client以其共同的Service Name連線，但各自的host所屬的 Instance 都有其特定的Instance Name，例如二個node的Instance Name分別為rac1 及rac2，但連線時以其Service Name連線，例如RAC。
1. SID(System Identifier)是唯一的，代表一個特定的資料庫；SERVICE_NAME可以關聯一個或多個SID。可以把SERVICE_NAME看做是SID的別名。連線至SERVICE_NAME的好處, 是當其背後代表的SID資料庫實體替換時，client不用再改動SERVICE_NAME的連線資訊，通常應用在有多個資料庫備援的cluster環境。
1. 連線目標Oracle的服務名稱(Service_Name)，通常是全域資料庫名稱(SID / Listener.GLOBAL_DBNAME(orcl.cko2x.tw))

---

:bulb: **注意!!**
:book: **注意!!**
<i class="fa fa-file-text"> 雜湊 Hash</i>
<i class="fa fa-file-text"> 附件</i>
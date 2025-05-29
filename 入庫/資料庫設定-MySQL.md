---
title: 資料庫設定-MySQL
tags: [入庫]

---

- [pymysql报错：cryptography is required for sha256_password or caching_sha2_password](https://blog.csdn.net/MK_chan/article/details/89390368)
- [MySQL 新增使用者及建立資料庫權限](https://www.ltsplus.com/mysql/mysql-add-new-users-databases-privileges)

---

### *- 建立開放遠端連線的帳號*

在 MySQL 8.0 的 command line 中，要建立允許遠端連線的帳號，你需要使用 `CREATE USER` 和 `GRANT` 來完成。以下是建立一個允許遠端連線的帳號的 command line 語法：

1. 首先，使用 root 或具有管理權限的帳號登入 MySQL：

```sql
mysql -u root -p
```

2. 接著，建立一個新的使用者（如果該使用者不存在）：

```sql
CREATE USER 'your_username'@'%' IDENTIFIED BY 'your_password';
```

這裡的 `your_username` 是你想要建立的使用者名稱，`your_password` 是你要設定的密碼。`'%'` 是允許該使用者從任何 IP 地址進行遠端連線。

3. 授予該使用者適當的權限，例如，如果你希望該使用者擁有所有資料庫的全部權限：

```sql
GRANT ALL PRIVILEGES ON *.* TO 'your_username'@'%' WITH GRANT OPTION;
```

4. 最後，記得重新載入權限設定：

```sql
FLUSH PRIVILEGES;
```

完成上述步驟後，你的帳號就可以從任何遠端位置連接到 MySQL 伺服器。請記得在真實環境中維護好安全性，僅開放必要的權限給遠端使用者。

---
title: DbContext.AddRange 比 DbContext.Add 更快？？
tags: [.Entity Framework Core, .Entity Framework]

---

1. **減少與數據庫的溝通次數**：
   - Add：每個實體都會產生一個獨立的 INSERT 語句
   - AddRange：Entity Framework 會將多個實體組合成一個批次操作

2. **記憶體管理效率**：
   - AddRange 內部使用了更優化的記憶體分配方式
   - 可以一次性分配所需的記憶體空間，而不是反覆分配

3. **實際效能差異**：
   ```csharp
   // 假設要新增 1000 筆資料
   
   // 使用 Add (較慢)
   for(int i = 0; i < 1000; i++) {
       context.Users.Add(new User()); 
   }
   
   // 使用 AddRange (較快)
   var users = new List<User>(1000);
   for(int i = 0; i < 1000; i++) {
       users.Add(new User());
   }
   context.Users.AddRange(users);
   ```

   在大量資料的情況下（如上述 1000 筆），AddRange 可能會比單筆 Add 快上 20-30%。

不過要注意：
- 如果只是新增少量資料（如 1-2 筆），差異可能不明顯
- 實際效能增益會依據資料量、資料庫設定、硬體等因素而有所不同
- SaveChanges() 的執行時間仍然是主要的效能考量點

所以建議：
- 當需要新增多筆資料時，優先使用 AddRange
- 當只需新增單筆資料時，使用 Add 即可
- 如果是批次處理大量資料，可以考慮使用 ==SQL Bulk Insert== 來獲得更好的效能
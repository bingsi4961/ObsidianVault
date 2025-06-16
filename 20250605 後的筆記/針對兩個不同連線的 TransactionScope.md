---
date: 2025-06-16 11:39
aliases: 
tags:
  - Entity_Framework
  - Entity_Framework_6
  - Entity_Framework_Core
---

# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
```csharp
try
{
    using (TransactionScope ts = new TransactionScope())
    {
        using (SqlConnection cn = new SqlConnection(Properties.Settings.Default.NW))
        {
            SqlCommand cmd = new SqlCommand("Insert Into Employees (LastName, FirstName) Values (@LastName, @FirstName)", cn);
            cmd.Parameters.AddWithValue("@LastName", textBox2.Text);
            cmd.Parameters.AddWithValue("@FirstName", textBox3.Text);

            cn.Open();
            int affectedRow = cmd.ExecuteNonQuery();
            MessageBox.Show($"新增 {affectedRow} 筆員工資料!!");
        }

        using (SqlConnection cn2 = new SqlConnection(Properties.Settings.Default.NW))
        {
            // 成功Test
            //SqlCommand cmd2 = new SqlCommand("Insert Into Products (ProductName, UnitPrice) Values ('xxx', 10)", cn2);
            // 失敗Test
            SqlCommand cmd2 = new SqlCommand("Insert Into Products (ProductName, UnitPrice) Values ('xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx', 10)", cn2);

            cn2.Open();
            int affectedRow = cmd2.ExecuteNonQuery();
            MessageBox.Show($"新增 {affectedRow} 筆產品資料!!");
        }

        ts.Complete();
        MessageBox.Show("交易成功!");
    }
}
catch (Exception)
{
    MessageBox.Show("交易失敗，已Rollback!!");
}
```
---
title: DB First、Model First、Code First
tags: [.Entity Framework]

---

# Code First 時，設定複合式主鍵

```csharp=

public class CinemaContext : DbContext
{
    public CinemaContext()
    {
        Database.SetInitializer<CinemaContext>(new DropCreateDatabaseAlways<CinemaContext>());
    }

    public DbSet<Genre> Genres { get; set; }
    public DbSet<Movie> Movies { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        //base.OnModelCreating(modelBuilder);
        
        // 當我們設定關聯時，需要遵循這樣的鏈式呼叫順序：
        // 先指定關係的類型（必要/選擇性）
        // 指定導航屬性的多重性（一對多/一對一等）
        // 最後才能指定外鍵

        modelBuilder.Entity<Genre>()
            .HasKey(g => new { g.GenreID, g.GenreID2 });

        modelBuilder.Entity<Movie>()
            .HasRequired(m => m.Genre)    // 指定關聯的必要性和導航屬性
            .WithMany(m => m.Movies)      // 指定關係的多重性（一對多）
            .HasForeignKey(m => new { m.GenreID, m.GenreID2 });    // // 最後指定外鍵
            
        // 假設 Movie 可以不屬於任何 Genre，那麼就改成如下
        // modelBuilder.Entity<Movie>()
        //     .HasOptional(m => m.Genre)  // 改用 HasOptional
        //     .WithMany(g => g.Movies)
        //     .HasForeignKey(m => new { m.GenreID, m.GenreID2 });
    }
}

public class Genre
{        
    public int GenreID { get; set; }            
    public int GenreID2 { get; set; }

    [Required]
    [StringLength(100)]
    public string GenreName { get; set; }

    public virtual ICollection<Movie> Movies { get; set; }
}


[Table("Movies1107")]   // 表示在資料庫中，表格名稱是 "Movies1107"
public class Movie
{
    [Key]
    public int ID { get; set; }

    [Required]
    [StringLength(100)]
    public string Title { get; set; }

    [StringLength(500)]
    public string Description { get; set; }

    // 代表資料庫中，欄位名稱是 "PublishDate"
    // 在 C# 類別中，我們用 PubDate 這個屬性名稱
    [Column("PublishDate")] 
    public Nullable<DateTime> PubDate { get; set; }
            
    public int GenreID { get; set; }
    public int GenreID2 { get; set; }
    
    // 假設 Movie 可以不屬於任何 Genre，那麼就改成如下
    // public int? GenreID { get; set; }
    // public int? GenreID2 { get; set; }    

    public virtual Genre Genre { get; set; }
}

```



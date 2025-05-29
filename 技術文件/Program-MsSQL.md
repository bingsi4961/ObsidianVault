---
title: Program-MsSQL
tags: [技術文件]

---

# Program-MsSQL
###### tags: `技術文件`

---
## 待確認
- [查詢效能調校](https://dotblogs.com.tw/ricochen/2011/07/31/32374)
- [查詢調校 — 計算的資料行](https://medium.com/ricos-note/sql-server-%E6%9F%A5%E8%A9%A2%E8%AA%BF%E6%A0%A1-%E8%A8%88%E7%AE%97%E7%9A%84%E8%B3%87%E6%96%99%E8%A1%8C-8f7eb2e19d0a)
- [善用計算的資料行](https://medium.com/ricos-note/sql-server-%E5%96%84%E7%94%A8%E8%A8%88%E7%AE%97%E7%9A%84%E8%B3%87%E6%96%99-dd7a46431fba)

---
## Singles
- [什麼是SET NOCOUNT ON](https://ithelp.ithome.com.tw/articles/10198514)
- [將一資料表複製](https://dotblogs.com.tw/mis0800/2014/02/09/143889)
- [Date and Time Conversions](https://www.mssqltips.com/sqlservertip/1145/date-and-time-conversions-using-sql-server/)
- [各式各樣的GETDATE()時間格式轉換CONVERT](https://www.dotblogs.com.tw/kevinya/2014/09/05/146474)
- [SQL指令筆記](https://blog.poychang.net/note-ms-sql/)
- [小心駛得萬年船--SQL指令保險栓](https://blog.darkthread.net/blog/safty-lock/)
- [Begin、Commit、RollBack Transaction](https://ithelp.ithome.com.tw/articles/10190252)
- [使用Transation機制確保資料異動正確性](https://dotblogs.com.tw/just_design/2017/08/12/013333)

---
## Performance Tuning
- [20條Tips方案，最佳化你的SQL查詢](https://www.finereport.com/tw/data-analysis/sqlquery.html)
- [Database — 八個 Index( 索引 ) 無法生效的 SQL 寫法](https://medium.com/johnliu-%E7%9A%84%E8%BB%9F%E9%AB%94%E5%B7%A5%E7%A8%8B%E6%80%9D%E7%B6%AD/database-%E5%85%AB%E5%80%8B-index-%E7%B4%A2%E5%BC%95-%E7%84%A1%E6%B3%95%E7%94%9F%E6%95%88%E7%9A%84-sql-%E5%AF%AB%E6%B3%95-cdc7d2e72f51)
- [簡化大量刪除資料的指令](https://dotblogs.com.tw/jamesfu/2015/05/17/sp_largedelete)
- [count function小提醒](https://dotblogs.com.tw/ricochen/2016/02/23/212205)
- [count function小提醒(下)](https://dotblogs.com.tw/ricochen/2016/05/08/223149)

---
## LOCK
- [MS SQL 鎖定 (Dead Lock)](https://bef0510.github.io/2019/07/11/MSSqlDeadLock/)
- [SQL-NOLOCK與ROWLOCK](https://dotblogs.com.tw/sunnylearning/2018/04/11/115205)
- [Will adding a nonclustered index lock my table?](https://stackoverflow.com/questions/22094482/will-adding-a-nonclustered-index-lock-my-table)
- [Create a SQL Server index without locking the table](https://rsw.io/create-a-sql-server-index-without-locking-the-table/)


---
## Example - Create Index
:::spoiler
```sql=
USE [SS2_DB]

--【變更前】--
SELECT * FROM SYS.INDEXES WHERE NAME = 'IX_TRANSDATE'

--【進行變更】--

-- 變更復原模式為簡單
ALTER DATABASE [SS2_DB] SET RECOVERY SIMPLE WITH NO_WAIT

DECLARE @Start DATETIME = GETDATE()
PRINT N'開始建立INDEX：' + (CONVERT(VARCHAR(24), @Start, 121))

CREATE INDEX IX_TRANSDATE ON translog (TRANSDATE);   
--WAITFOR DELAY N'00:00:05.100'

PRINT N'INDEX建立完成：' + (CONVERT(VARCHAR(24), GETDATE(), 121))
PRINT N'執行時間為:' + CAST(DATEDIFF(ss, @Start, GETDATE()) AS VARCHAR(10)) + N'秒'

-- 變更復原模式為完整
ALTER DATABASE [SS2_DB] SET RECOVERY FULL WITH NO_WAIT

--【變更後】--
SELECT * FROM SYS.INDEXES WHERE NAME = 'IX_TRANSDATE'
```
:::

---
## Example - Delete Rows
:::spoiler
```sql=
USE [SS2_DB]

-- 變更復原模式為簡單
ALTER DATABASE [SS2_DB] SET RECOVERY SIMPLE WITH NO_WAIT

DECLARE @Start DATETIME               -- 執行開始時間
DECLARE @DelRowCount INT = 10000	-- 單次刪除筆數
DECLARE @Count INT = -1		-- 單次影響筆數
DECLARE @Total INT = 0			-- 刪除總筆數
DECLARE @Delete NVARCHAR(40)		-- 刪除總筆數描述
DECLARE @BeforeDelete NVARCHAR(30)	-- 刪除前資料表總筆數描述
DECLARE @AfterDelete NVARCHAR(30)	-- 刪除後資料表總筆數描述
DECLARE @DelRowDate DATETIME		-- 刪除哪天前的資料

SET NOCOUNT ON

--【變更前】--
SET @BeforeDelete = N'刪除前總筆數:' + CAST((SELECT COUNT(*) FROM translog WITH (NOLOCK)) AS VARCHAR(10))
PRINT @BeforeDelete

SET @Start = GETDATE()
SET @DelRowDate = DATEADD(YEAR, -5, GETDATE())

--【進行變更】--
WHILE @Count <> 0
BEGIN
	--BEGIN TRAN
		DELETE TOP (@DelRowCount) FROM translog WHERE TRANSDATE < @DelRowDate OPTION (MAXDOP 1)		
		SET @Count = @@ROWCOUNT
		SET @Total = @Total + @Count
	--COMMIT	
	--IF (@Total % (@DelRowCount * 50)) = 0 DBCC SHRINKFILE (N'SS2_DB_Log')
	IF (@Total % (@DelRowCount * 50)) = 0 
	BEGIN
		WAITFOR DELAY N'00:00:02.000'
		DBCC SHRINKFILE (N'SS2_DB_Log')
	END
END

SET @Delete = N'刪除筆數:' + CAST(@Total AS VARCHAR(10)) + N', 執行時間:' + CAST(DATEDIFF(ss, @Start, GETDATE()) AS VARCHAR(10)) + N'秒'
PRINT @Delete

--【變更後】--
SET @AfterDelete = N'刪除後總筆數:' + CAST((SELECT COUNT(*) FROM translog WITH (NOLOCK)) AS VARCHAR(10))
PRINT @AfterDelete

SET NOCOUNT OFF

-- 變更復原模式為完整
ALTER DATABASE [SS2_DB] SET RECOVERY FULL WITH NO_WAIT
GO
```
:::

---
## Example - Delete Rows 2
:::spoiler
```sql=
USE [SS2_DB]

-- 變更復原模式為簡單
ALTER DATABASE [SS2_DB] SET RECOVERY SIMPLE WITH NO_WAIT

DECLARE @Start DATETIME		-- 執行開始時間
DECLARE @DelRowCount INT = 10000	-- 單次刪除筆數
DECLARE @Count INT = -1		-- 單次影響筆數
DECLARE @Total INT = 0			-- 刪除總筆數
DECLARE @Delete NVARCHAR(40)		-- 刪除總筆數描述
DECLARE @BeforeDelete NVARCHAR(30)	-- 刪除前資料表總筆數描述
DECLARE @AfterDelete NVARCHAR(30)	-- 刪除後資料表總筆數描述
DECLARE @DelRowDate NVARCHAR(30)	-- 刪除哪天前的資料
DECLARE @DelCommand NVARCHAR(300)	-- 刪除資料SQL

SET NOCOUNT ON

SET @Start = GETDATE()
SET @DelRowDate = CONVERT(NVARCHAR(30),DATEADD(YEAR, -5, GETDATE()),121)
SET @DelCommand = 'DELETE TOP (' + CAST(@DelRowCount AS NVARCHAR(5)) + ') FROM translog WHERE TRANSDATE < ''' + @DelRowDate + ''' OPTION (MAXDOP 1)'

--【變更前】--
SET @BeforeDelete = N'刪除前總筆數:' + CAST((SELECT COUNT(*) FROM translog WITH (NOLOCK)) AS VARCHAR(10))
PRINT @BeforeDelete

--【進行變更】--
WHILE @Count <> 0
BEGIN
	--BEGIN TRAN
		--DELETE TOP (@DelRowCount) FROM translog WHERE TRANSDATE < @DelRowDate OPTION (MAXDOP 1)
		EXEC SP_EXECUTESQL @DelCommand
		SET @Count = @@ROWCOUNT
		SET @Total = @Total + @Count
	--COMMIT	
	--IF (@Total % (@DelRowCount * 50)) = 0 DBCC SHRINKFILE (N'SS2_DB_Log')
	IF (@Total % (@DelRowCount * 50)) = 0 
	BEGIN
		WAITFOR DELAY N'00:00:02.000'
		DBCC SHRINKFILE (N'SS2_DB_Log')
	END	
END

SET @Delete = N'刪除筆數:' + CAST(@Total AS VARCHAR(10)) + N', 執行時間:' + CAST(DATEDIFF(ss, @Start, GETDATE()) AS VARCHAR(10)) + N'秒'
PRINT @Delete

--【變更後】--
SET @AfterDelete = N'刪除後總筆數:' + CAST((SELECT COUNT(*) FROM translog WITH (NOLOCK)) AS VARCHAR(10))
PRINT @AfterDelete

SET NOCOUNT OFF

-- 變更復原模式為完整
ALTER DATABASE [SS2_DB] SET RECOVERY FULL WITH NO_WAIT
GO
```
:::

---
## Example - 將表格1的資料填入表格2、Shrink
:::spoiler
```sql=
USE SS2_DB;
GO

-- Truncate the log by changing the database recovery model to SIMPLE.
ALTER DATABASE SS2_DB SET RECOVERY SIMPLE;
GO

-- 刪除表格、新建表格，並從舊表格撈資料塞進新表格、建立索引
DROP TABLE translog_n2
SELECT * INTO translog_n2 FROM translog OPTION (MAXDOP 1)
CREATE INDEX "IDX_TRANSDATE_N2" ON translog_n2 (TRANSDATE)

-- 清除新表格資料，並從舊表格撈資料塞進新表格
TRUNCATE TABLE translog_n2
INSERT INTO translog_n2 SELECT * FROM translog

-- Shrink DataBase
-- Shrink the truncated log file to 1 MB.
DBCC SHRINKDATABASE(N'SS2_DB')
DBCC SHRINKFILE (SS2_DB_Log, 1);
GO

-- Reset the database recovery model.
ALTER DATABASE SS2_DB SET RECOVERY FULL;
GO
```
:::

---
## Example - Create Stored Procedure
:::spoiler
```sql=

USE [SS2_DB]
GO

/****** Object:  StoredProcedure [dbo].[spHouseKeepTranslog]    Script Date: 2023/1/19 下午 03:37:14 ******/
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO


CREATE PROCEDURE [dbo].[spHouseKeepTranslog] AS 
BEGIN
	
	DECLARE @DelRowCount INT = 100		-- 單次刪除筆數
	DECLARE @ExeCount INT = 0	        -- 指令執行次數	
	DECLARE @DelRowDate NVARCHAR(30)	-- 刪除哪天前的資料
	DECLARE @DelCommand NVARCHAR(300)	-- 刪除資料SQL

	SET NOCOUNT ON
	
	SET @DelRowDate = CONVERT(NVARCHAR(30),DATEADD(YEAR, -5, GETDATE()),121)
	SET @DelCommand = 'DELETE TOP (' + CAST(@DelRowCount AS NVARCHAR(3)) + ') FROM translog WHERE TRANSDATE < ''' + @DelRowDate + ''' OPTION (MAXDOP 1)'

	WHILE @ExeCount <= 14
	BEGIN		
		SET @ExeCount = @ExeCount + 1		
		BEGIN TRY
			EXEC SP_EXECUTESQL @DelCommand			
		END TRY
		BEGIN CATCH
			-- N/A
		END CATCH
	END

	SET NOCOUNT OFF
END
GO

```
:::
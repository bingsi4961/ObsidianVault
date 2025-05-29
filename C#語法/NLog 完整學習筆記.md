---
title: NLog 完整學習筆記
tags: ['C# 程式']

---

# 一、NLog 核心概念與架構

## 三大核心組件

**Logger（記錄器）**：負責接收你的日誌訊息，就像郵遞系統中的郵筒。

**Target（目標）**：定義訊息要輸出到哪裡，例如檔案、控制台、資料庫等。就像郵件的目的地地址。

**Rule（規則）**：決定什麼樣的訊息要送到哪個 Target，就像郵遞系統的分發規則。

# 二、安裝與基本設定

## NuGet 套件安裝

**基本 .NET Framework 專案**：
```
Install-Package NLog
```

**ASP.NET Core 專案**（推薦）：
```
Install-Package NLog.Web.AspNetCore
```

**為什麼選擇 NuGet**：
- 自動處理相依性管理，確保 NLog 需要的其他套件也會一併安裝。
- 簡化版本控制，輕鬆升級或降級到特定版本。
- 確保安裝官方穩定版本

## NLog.config 檔案設定

**重要提醒**：NLog.config 檔案必須正確設定檔案屬性：
- 「建置動作」設為 **Content**
- 「複製到輸出目錄」設為 **有更新時才複製** 或 **永遠複製**

這些設定確保 NLog.config 會被複製到輸出目錄，讓應用程式能夠在執行時找到設定檔。

# 三、日誌等級階層（從低到高）

## 六個日誌等級的理解

**Trace（等級值：0）**：最詳細的訊息，程式的「內心獨白」，記錄每一個細微的步驟。生產環境通常關閉。

**Debug（等級值：1）**：開發階段的除錯資訊，幫助理解程式執行流程。像是程式設計師在程式碼中的「筆記」。

**Info（等級值：2）**：一般性的資訊訊息，記錄程式的正常運作流程。告訴你「程式正在做什麼」，是生產環境最常見的等級。

**Warn（等級值：3）**：警告訊息，表示可能有問題但程式仍能繼續執行。就像汽車儀表板的警示燈。

**Error（等級值：4）**：錯誤訊息，表示發生了問題但程式可能還能繼續運作。

**Fatal（等級值：5）**：嚴重錯誤，通常會導致程式終止。這是最嚴重的問題等級。

## 等級篩選原理

* `minlevel` 和 `maxlevel` 定義了日誌等級的範圍。levels 屬性讓你指定確切的等級清單。
* 當設定 `minlevel="Info"` 時，NLog 會記錄 Info、Warn、Error 和 Fatal 等級的訊息，但會忽略 Trace 和 Debug 等級的訊息。

# 四、Logger 名稱機制

## Logger 名稱的來源

logger 時的 name 是什麼。這個問題的答案取決於你建立 Logger 實例的方式。

#### 最常見的方式：GetCurrentClassLogger

當你使用 `LogManager.GetCurrentClassLogger()` 時，Logger 的名稱會自動設定為呼叫這個方法的類別的完整名稱（包含命名空間）。讓我用一個具體的範例來說明：

```csharp
// 假設這個類別位於 MyApplication.Services 命名空間中
namespace MyApplication.Services
{
    public class UserService
    {
        // 這個 Logger 的名稱會是 "MyApplication.Services.UserService"
        private static readonly Logger logger = LogManager.GetCurrentClassLogger();
        
        public void CreateUser(string username)
        {
            // 當你呼叫這行時，Logger 的名稱就是 "MyApplication.Services.UserService"
            logger.Info("正在建立使用者: {Username}", username);
        }
    }
}
```

#### 手動指定 Logger 名稱

```csharp
// 手動指定 Logger 名稱為 "OrderProcessing"
private static readonly Logger logger = LogManager.GetLogger("OrderProcessing");

// 或者使用更具描述性的名稱
private static readonly Logger auditLogger = LogManager.GetLogger("Audit.OrderOperations");
```

## 實際驗證 Logger 名稱的方法

```csharp
// 在日誌訊息中顯示 Logger 名稱
logger.Info("Logger 名稱是: {LoggerName}", logger.Name);

// 或者在 NLog.config 的 layout 中使用 ${logger} 變數
// 這樣每條日誌都會自動包含 Logger 名稱
```

# 五、實際程式碼使用方法

## 基本日誌記錄語法

**Trace 使用**：記錄詳細的執行流程
```csharp
_logger.Trace("進入方法，參數值：{ParameterValue}", parameterValue);
```

**Debug 使用**：開發階段的除錯資訊
```csharp
_logger.Debug("開始處理資料，項目數量：{ItemCount}", items.Count);
```

**Info 使用**：記錄重要的業務事件
```csharp
_logger.Info("使用者登入成功 - UserId: {UserId}, LoginTime: {LoginTime}", userId, DateTime.Now);
```

**Warn 使用**：記錄警告但不阻止程式繼續的情況
```csharp
_logger.Warn("外部服務回應緩慢，耗時：{ResponseTime}ms", responseTime);
```

**Error 使用**：記錄錯誤並包含例外詳細資訊
```csharp
try
{
    // 可能出錯的程式碼
}
catch (Exception ex)
{
    _logger.Error(ex, "處理訂單時發生錯誤 - OrderId: {OrderId}", orderId);
    throw;
}
```

**Fatal 使用**：記錄可能導致系統無法繼續的嚴重錯誤
```csharp
_logger.Fatal(ex, "資料庫連線初始化失敗，應用程式無法啟動");
```

## 結構化參數的重要性

**推薦方式**（結構化參數）：
```csharp
_logger.Info("使用者操作 - UserId: {UserId}, Action: {Action}", userId, action);
```

**不推薦方式**（字串插值）：
```csharp
_logger.Info($"使用者操作 - UserId: {userId}, Action: {action}");
```

**結構化參數的優勢**：保留了資料的原始型別資訊，讓日誌分析工具能夠更有效地索引和搜尋。

# 六、NLog.config 設定詳解

## 根節點重要屬性

```xml
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Info"
      internalLogFile="logs/nlog-internal.log">
```

**autoReload="true"**：設定檔變更時自動重新載入，無需重啟應用程式。

**throwExceptions="false"**：即使 NLog 本身有問題，也不會影響主程式運作。

**internalLogLevel 和 internalLogFile**：NLog 自我診斷工具，幫助排查日誌系統問題。

## Targets 設定詳解

### 檔案目標完整設定

```xml
<!-- 定義 Target，也就是日誌要輸出到哪裡 -->
<target xsi:type="File" 
        name="fileTarget"
        fileName="${basedir}/logs/${shortdate}.log"
        layout="${longdate} ${level:uppercase=true} ${logger} ${message} ${exception:format=tostring}"
        archiveFileName="${basedir}/logs/archive/{#}.log"
        archiveEvery="Day"
        archiveNumbering="Rolling"
        maxArchiveFiles="30"
        archiveAboveSize="10485760"
        encoding="utf-8"
        deleteOldFileOnStartup="false"
        createDirs="true"
        concurrentWrites="true"
        keepFileOpen="false">
</target>
```

**檔案目標所有屬性詳細說明**：

**xsi:type="File"**：指定這是檔案類型的目標。這個屬性告訴 NLog 這個目標會將日誌寫入檔案系統。就像是告訴郵遞員這個包裹要送到實體地址而不是電子信箱。

**name="fileTarget"**：目標的唯一識別名稱，用於在 Rules 中引用這個目標。選擇有意義的名稱讓設定更易於理解和維護。這就像是為每個目標貼上標籤，方便之後參照。

**fileName="\${basedir}/logs/\${shortdate}.log"**：指定日誌檔案的完整路徑和檔案名稱。支援 NLog 變數和日期格式化。這個屬性定義了日誌的實際存放位置，就像是信件的收件地址。範例中會產生如 `C:\MyApp\logs\2025-05-27.log` 這樣的檔案。

**layout="${longdate} ${level:uppercase=true} ${logger} ${message} ${exception:format=tostring}"**：定義每條日誌記錄的格式模板。這個屬性控制日誌在檔案中的呈現方式，包含時間戳記、日誌等級、來源類別、訊息內容和例外資訊。你可以把它想像成信件的格式範本，決定收件人看到的內容排列方式。

**archiveFileName="${basedir}/logs/archive/{#}.log"**：定義歸檔檔案的命名模式。`{#}` 是一個特殊的佔位符，會被實際的檔案編號替換。當現有日誌檔案需要歸檔時，會依照這個模式重新命名並移動到指定位置。這就像是整理舊信件的歸檔系統。

**archiveEvery="Day"**：定義歸檔的觸發條件。可選值包括 Day（每日）、Hour（每小時）、Minute（每分鐘）、Month（每月）、Year（每年）、Sunday 到 Saturday（指定星期幾）。這個設定決定了多久創建一個新的日誌檔案。想像這是你決定多久整理一次文件櫃的規則。

**archiveNumbering="Rolling"**：歸檔檔案的編號策略。Rolling 表示重複使用編號(從1到10)，最新的歸檔檔案編號最小。Sequence 表示遞增編號永不重複。Date 表示使用日期時間作為檔案名稱的一部分。Rolling 就像是有限的停車位，新車來了舊車就要離開；Sequence 像是銀行排號，永遠遞增。

**maxArchiveFiles="30"**：保留的歸檔檔案數量上限。超過這個數量時，最舊的檔案會被自動刪除。這個設定幫助控制磁碟空間使用量。就像是相片收藏盒，只保留最近的 30 張相片，舊的會被移除。

**archiveAboveSize="10485760"**：當現有日誌檔案超過指定大小（位元組）時觸發歸檔。10485760 位元組等於 10MB。這個機制確保單一日誌檔案不會變得過大而難以處理。想像這是一個垃圾桶，裝滿了就要清空換新的。

**encoding="utf-8"**：指定檔案的字元編碼格式。UTF-8 是現代應用程式的標準編碼，支援各種語言字元。這確保中文、日文或其他特殊字元能正確顯示。就像是選擇正確的翻譯字典，確保所有文字都能正確理解。

**deleteOldFileOnStartup="false"**：決定應用程式啟動時是否刪除現有的日誌檔案。設為 true 在開發環境很有用，可以每次都有乾淨的開始。設為 false 在生產環境很重要，避免遺失重要的歷史日誌。這就像是決定搬家時是否要丟掉舊家具。

**createDirs="true"**：當指定的目錄路徑不存在時，是否自動創建整個目錄結構。設為 true 讓 NLog 具有自我管理能力，不需要手動創建資料夾。這就像是一個貼心的助手，會自動準備好所需的整理空間。

**concurrentWrites="true"**：允許多個程序或執行緒同時寫入同一個日誌檔案。在多執行緒應用程式或多個應用程式實例共享日誌檔案時很重要。這就像是允許多人同時在同一本記事本上寫筆記，但要確保不會相互干擾。

**keepFileOpen="false"**：控制是否保持日誌檔案持續開啟。設為 false 表示每次寫入後關閉檔案，這讓其他程式能夠讀取或移動檔案，但會稍微影響寫入效能。設為 true 則保持檔案開啟以提升效能，但檔案會被鎖定。這是在效能和資源可用性之間的權衡選擇。

### 其他常用目標

**控制台目標**：
```xml
<target xsi:type="Console" 
        name="consoleTarget"
        layout="${time} [${level}] ${message}"
        detectConsoleAvailable="true" />
```

**資料庫目標**：
```xml
<target xsi:type="Database"
        name="databaseTarget"
        connectionString="Server=localhost;Database=LogDB;Integrated Security=true"
        commandText="INSERT INTO Logs (Date, Level, Logger, Message, Exception) VALUES (@date, @level, @logger, @message, @exception)">
  <parameter name="@date" layout="${date}" />
  <parameter name="@level" layout="${level}" />
  <parameter name="@logger" layout="${logger}" />
  <parameter name="@message" layout="${message}" />
  <parameter name="@exception" layout="${exception:tostring}" />
</target>
```

### 非同步寫入
```xml
<targets async="true">
  <target xsi:type="File" name="asyncFile" />
</targets>
```

## Rules 執行機制

### 順序執行原理

Rules 按照定義順序執行，每條日誌訊息會依序檢查每個規則。如果規則匹配，日誌會被寫入對應的 Target。**重要**：一條日誌可能匹配多個規則，因此會被寫入多個目標。

```xml
<rules>
  <!-- 規則會按順序執行 -->
  
  <!-- 記錄所有 Error 和 Fatal 等級的日誌到錯誤檔案，適用於所有命名空間(*) -->
  <logger name="*" levels="Error,Fatal" writeTo="errorFile" />
  
  <!-- 記錄 MyApp.Services 命名空間下的服務類別日誌 -->
  <!-- 從 Info 等級開始記錄，包含 Info, Warn, Error, Fatal -->
  <logger name="MyApp.Services.*" minlevel="Info" writeTo="serviceFile" />
  
  <!-- 記錄所有其他的一般日誌 -->
  <!-- 適用於所有命名空間(*)，從 Info 等級開始記錄到一般檔案 -->
  <logger name="*" minlevel="Info" writeTo="generalFile" />
</rules>
```

### final 屬性的重要性

```xml
<!-- 特殊處理：Microsoft 相關的日誌只記錄到特定檔案 -->
<logger name="Microsoft.*" minlevel="Warn" writeTo="microsoftFile" final="true" />
```

`final="true"` 表示如果這個規則匹配，就停止檢查後續規則。這用於阻止某些日誌繼續流向其他目標。

### 萬用字元匹配

**星號 (*) 用法**：
- `*`：匹配所有 Logger 名稱
- `MyApp.Services.*`：匹配所有以 `MyApp.Services.` 開頭的 Logger
- `Microsoft.*`：匹配所有 Microsoft 框架產生的日誌

## 支援變數定義

```xml
<variable name="logDirectory" value="C:/Logs" />
<variable name="applicationName" value="MyApplication" />

<targets>
  <target xsi:type="File" 
          name="fileTarget"
          fileName="${var:logDirectory}/${var:applicationName}-${shortdate}.log" />
</targets>
```


## 內建變數活用

### basedir 變數

`${basedir}` 代表應用程式的基礎目錄，確保設定檔的可攜性。無論應用程式部署在哪個環境，都能自動找到正確的基礎路徑。

```xml
<target fileName="${basedir}/logs/${shortdate}.log" />
```

### 其他實用變數

**日期變數**：
- `${shortdate}`：yyyy-MM-dd 格式
- `${date:format=yyyy-MM}`：自訂日期格式

**環境變數整合**：
```xml
<target fileName="${environment:LOGS_PATH:whenEmpty=${basedir}/logs}/${shortdate}.log" />
```
先嘗試讀取 LOGS_PATH 環境變數，若不存在則使用預設路徑。

### 進階目錄組織

```xml
<target fileName="${basedir}/logs/${date:format=yyyy}/${date:format=MM}/${level}/${shortdate}.log" />
```
創造階層式目錄結構：logs/2025/05/Info/2025-05-27.log

# 七、Microsoft 框架日誌的橋接機制

Microsoft 框架本身不直接呼叫 NLog，而是透過 `Microsoft.Extensions.Logging` 抽象層。當你在 `Program.cs` 中設定 `builder.Host.UseNLog()` 時，實際上是註冊 NLog 作為日誌提供者，所有框架產生的日誌會自動轉發給 NLog 處理。

這就是為什麼你會看到類似 `Microsoft.EntityFrameworkCore.Database.Command` 這樣的 Logger 名稱，它們來自框架內部的類別。

**Program.cs 設定：**

```csharp
using NLog.Web;

var builder = WebApplication.CreateBuilder(args);

// 移除預設日誌提供者(ILogger)並新增 NLog
builder.Logging.ClearProviders();
builder.Host.UseNLog();

var app = builder.Build();
app.Run();
```

# 八、自動記錄所有未處理的例外

N/A

# 九、實務應用策略

## 日誌分離策略

**按等級分離**：將錯誤日誌和一般日誌分開存放，便於快速定位問題。

**按模組分離**：根據命名空間將不同功能模組的日誌分開記錄。

**框架日誌隔離**：將 Microsoft 框架產生的大量技術性日誌與應用程式業務日誌分開。

## 環境配置策略

**開發環境**：
- 較低的日誌等級（Debug、Trace）
- 控制台輸出便於即時查看
- 詳細的框架日誌

**生產環境**：
- 較高的日誌等級（Info 以上）
- 檔案輸出並設定歸檔
- 過濾框架噪音日誌

# 十、常見問題與解決方案

## 設定檔案問題

**問題**：應用程式找不到 NLog.config
**解決**：確認檔案屬性設定為 Content 且複製到輸出目錄

**問題**：autoReload 功能不生效
**解決**：檢查檔案是否為 Embedded Resource（應該是 Content）

## 日誌輸出問題

**問題**：沒有看到預期的日誌檔案
**解決**：
1. 檢查日誌等級設定
2. 確認 Rules 匹配邏輯
3. 檢查檔案權限和路徑存取

**問題**：日誌重複記錄
**解決**：檢查 Rules 設定，適當使用 final="true"

## 效能問題

**問題**：大量日誌影響應用程式效能
**解決**：
1. 使用非同步目標
2. 調整日誌等級
3. 設定適當的歸檔策略

# 十一、最佳實務建議

## 設定檔組織

1. **使用變數**：善用 ${basedir} 等變數確保可攜性
2. **環境區分**：不同環境使用不同的日誌策略
3. **定期清理**：設定適當的歸檔和清理策略
4. **監控空間**：監控日誌檔案的磁碟使用量
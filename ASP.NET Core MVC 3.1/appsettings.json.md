---
title: appsettings.json
tags: [ASP.NET Core MVC 3.1]

---

# 取得 appsettings.json 的值

1. 通過 `IConfiguration` 注入（最直接的方式）：
```csharp
// Controller 中使用
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }

    public IActionResult Index()
    {
        // 直接取值
        string value = _config["Key"];
        
        // 取巢狀值
        string nestedValue = _config["Parent:Child"];
        
        // 取得並轉型
        int intValue = _config.GetValue<int>("NumberKey");
        
        return View();
    }
}
```

2. 使用強型別設定類（比較推薦的方式）：
```csharp
// 1. 建立設定類別
public class MySettings
{
    public string SiteName { get; set; }
    public int MaxItems { get; set; }
    public string ApiUrl { get; set; }
}

// 2. 在 Startup.cs 註冊
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<MySettings>(Configuration.GetSection("MySettings"));
}

// 3. 在 Controller 使用
public class HomeController : Controller
{
    private readonly MySettings _settings;

    public HomeController(IOptions<MySettings> settings)
    {
        _settings = settings.Value;
    }

    public IActionResult Index()
    {
        string siteName = _settings.SiteName;
        return View();
    }
}
```

對應的 appsettings.json：
```json
{
  "MySettings": {
    "SiteName": "My Website",
    "MaxItems": 100,
    "ApiUrl": "http://api.example.com"
  }
}
```
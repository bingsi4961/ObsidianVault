---
title: ASP.Net MVC 雜記
tags: [技術文件]

---

# ASP.Net MVC 雜記
###### tags: `技術文件`

---
## 路由器

### MVC5

#### MVC Web

:::info
《Global.asax.cs》
```csharp=
RouteConfig.RegisterRoutes(RouteTable.Routes);   
```
  
《RouteConfig.cs》
```csharp=
routes.MapMvcAttributeRoutes();

routes.MapRoute(
    name: "OperaTitleRoute",
    url: "Opera/Title/{title}",
    defaults: new { controller = "Opera", action = "DetailsByTitle", title = UrlParameter.Optional }
);

routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
);
```
:::

#### WebAPI

:::warning
《Global.asax.cs》
```csharp=
GlobalConfiguration.Configure(WebApiConfig.Register);
```

《WebApiConfig.cs》
```csharp=
config.MapHttpAttributeRoutes();

// 這個路由會將 "api/custom/123" 導向 SpecialNameController 的 Get action。
config.Routes.MapHttpRoute(
    name: "CustomRoute",
    routeTemplate: "api/custom/{id}",
    defaults: new { controller = "SpecialName", action = "Get", id = RouteParameter.Optional }
);

config.Routes.MapHttpRoute(
    name: "CustomApi",
    routeTemplate: "api/custom/{controller}/{action}/{id}",
    defaults: new { id = RouteParameter.Optional }
);

config.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
```
:::

### .Net Core 3.1 MVC

#### MVC Web

```sharp=
《Startup.cs》
public void Configure(IApplicationBuilder app,.....) {	
    app.UseEndPoints(endpoints => {
        
        endpoints.MapControllerRoute(
            name:"default",
            pattern:"{controller=GamingInfo}/{action=GameExhibition}/{id?}"
        );
        
        endpoints.AddRazorPages();
    });	
}
```

#### WebAPI

```sharp=
<<Startup.cs>>
public void Configure(...)
{
    app.UseEndPoints(endpoints => {
        // 啟用屬性路由
        endpoints.MapControllers();
    });	
}
```

---
## Framework & Core 通用

```
/Views/_ViewImports.cshtml    // 檢視匯入	=> @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
/Views/_ViewStart.cshtml        // 檢視開始	=> @{ Layout = "_Layout"; }
/Views/Shared/_Layout.cshtml    // 版面配置
```

===============
QueryString
RouteData
FormData
類別屬性預設值

==================================================

回應標頭 Content-Type => text/plain;charset=utf-8
回應標頭 Content-Type => text/xml
回應標頭 Content-Type => text/html
回應標頭 Content-Type => application/json;charset=utf-8

---
## .Net Framework MVC5

N/A


---
## .Net Core MVC

:::info
// 在套件管理器主控台下指令

Scaffold-DbContext "Data Source=APZA002GOD;Initial Catalog=GamingPortalTest;User ID=DataOwner;Password=asus#1234;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models\DB -Context GamingPortalContext -Force
:::

```cshap=
《GamingPortalContext.cs》

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
    .LogTo(Console.WriteLine, Microsoft.Extensions.Logging.LogLevel.Information)        
    .UseSqlServer("Data Source=.\\sqlexpress; Initial Catalog=Northwind; Integrated Security=True; Encrypt=True; TrustServerCertificate=True;");
}

```

```csharp=
// .Net 7 以上版本
============= Program.cs ================

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllersWithViews();	// 註冊MVC服務

// 註冊 DbContext
builder.Services.AddDbContext<OperaContext>(options => options.UseSqlServer(ConnStr));
 
// 以上是註冊服務
// =================================================
// 以下是 request pipeline (請求管線) (由 MiddleWare Component 組成)

var app = builder.Build(); 

if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();    // 註冊 Https服務
app.UseStaticFiles()         // 靜態檔案(wwwroot)
app.UseRouting();
app.UseAuthorization(); 

app.MapControllerRoute(
	name:"default",
	pattern:"{controller=Home}/{action=Index}/{id?}"
);

app.MapGet("/",()=> "Hello World");
app.Run();
```

```json=
======= appSetting.json =======

{
    "ConnectionStrings":{
        "DefaultConnection":"Data Source=DESKTOP-23R0NPB;Initial Catalog=OperaDB;Integrated Security=True;Encrypt=True;TrustServerCertificate=True;MultipleActiveResultSets=True;",
        "DevConnection":"Data Source=DESKTOP-23R0NPB;Initial Catalog=OperaDB;Integrated Security=True;Encrypt=True;TrustServerCertificate=True;MultipleActiveResultSets=True;",
        "ProdConnection":"Data Source=DESKTOP-23R0NPB;Initial Catalog=OperaDB;Integrated Security=True;Encrypt=True;TrustServerCertificate=True;MultipleActiveResultSets=True;"
	},
    "Logging":{xxx},
    "AllowedHosts":"*"	
}

// 在 Program.cs 註冊 DbContext 服務時 =>
builder.Services.AddDbContext<OperaContext>(
    options=>options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")
	)
);

```

```CSharp=
public class SWGroupController : Controller 
{
	readonly IConfiguration _config;
	readonly string _ApiUrl;
	
	public SWGroupController(IConfiguration config){
		this._config = config;
		this._ApiUrl = this._config.GetValue<string>("ApiUrl");
	}
}

======= appsetting.json =======
{
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  
  "ConnectionStrings": {
    "Portal": "9v2DSIAOxKoM/qm0jQs2Yk3XkFt5powpzKezw5YX9pgOgx0OPmK64IfGgbj4nijKqA2ovxzgY/F6izADDUxVMoc3G95EV5c8KPp8BQIVxBiFyZ4C6/Vp9FJAefUozBjtYpHyqAn1TofjyaXayjUwpc81baxfJt0kskgw1bhvUr8="
  },
  
  "ApiUrl": "https://apza002gvc.corpnet.asus:8080/GamingAPI_Dev/api/",  
  "UploadFolder": "D:\\Upload\\Dev\\"
}
```



```html=
<form asp-controller="Home" asp-action="Submit" method="post">
<form action="/Home/Submit" method="post"> 

<form asp-action="Index" method="post" class="form-inline">
<form method="post" class="form-inline" action="/SysPost/Index">

<input asp-for="Title">
<input type="text" 
    data-val="true" 
    data-val-length="length error message" 
    data-val-required="rquired error message"
    data-val-length-max="200"
    maxlength="200"
    id="Title" name="Title" value="" />
會依據 Model 的 Title 屬性設定，生成 name="Title"、id="Title" 等... 如此在表單提交時，才能夠 Model Binding。
    
<span asp-validation-for="Title">*</span>
<span class="field-validation-valid" data-valmsg-for="Title" data-valmsg-replace="false">*</span>
class => 驗證成功時 field-validation-valid，驗證失敗時 field-validaation-error
data-valmsg-replace="false", 錯誤訊息使用*符號
data-valmsg-replace="true", 錯誤訊息使用 input text 所設定的錯誤訊息
    
<div asp-validation-summary="All"></div>
<div class="validation-summary-valid" data-valmsg-summary="true"></div>
class => 驗證成功時 validation-summary-valid，驗證失敗時 validation-summary-error
    
<div asp-validation-summary="All"></div>    .validation-summary-error(color:red)、.validation-summary-valid(display:none)
<label asp-for="Title"></label>
<input asp-for="Title" />                    .input-validation-error(color:red)
<span asp-validation-for="Title">            .field-validation-error(color:red)、.field-validation-valid(display:none)
    
    

    
<!-- 舊的寫法 -->
<script src="@Url.Content("~/js/site.js")" asp-append-version="true"></script>
<link href="@Url.Content("~/css/site.css")" rel="stylesheet" asp-append-version="true"/>

<!-- 建議的寫法 -->
<script asp-src="~/js/site.js" asp-append-version="true"></script>
<link asp-href="~/css/site.css" rel="stylesheet" asp-append-version="true"/>
    
<!-- 使用 asp-src-include 引入多個 js 檔案 -->
<script asp-src-include="~/js/**/*.js" asp-append-version="true"></script>

<!-- 使用 asp-href-include 引入多個 css 檔案 -->
<link rel="stylesheet" asp-href-include="~/css/**/*.css" asp-append-version="true" />

<!-- 排除特定檔案 -->
<script asp-src-include="~/js/**/*.js"
        asp-src-exclude="~/js/excluded/*.js"
        asp-append-version="true"></script>
    
** 匹配任意數量的目錄
* 匹配單一層級的檔案或目錄
; 分隔多個包含模式
    
<!-- 只包含特定檔案 -->
<script asp-src-include="~/js/main.js;~/js/utils.js"
        asp-append-version="true"></script>

<!-- 包含特定目錄下的檔案 -->
<link rel="stylesheet" 
      asp-href-include="~/css/shared/*.css"
      asp-append-version="true" />
    
<script asp-src-include="~/js/core/*.js;~/js/plugins/*.js"
        asp-src-exclude="~/js/plugins/debug/*.js"
        asp-append-version="true"></script>
```


---
## 未分

- [C# 10 的全域引用](https://www.huanlintalk.com/2022/02/csharp10-global-using.html)

==========================================

未指定 [FromBody] 或 [FromQuery]：

1. 基本數據類型參數：ASP.NET Core 會嘗試從查詢字串或表單數據中綁定。(先抓表單，若沒有再抓QueryString)
1. 複雜類型參數：默認會從表單數據中綁定，如果請求的內容是 application/x-www-form-urlencoded 或 multipart/form-data。
1. JSON 請求體：必須使用 [FromBody] 來綁定 JSON 請求體到複雜類型參數。(Content-Type: application/json)

==========================================

```cshar=
// 若 expression 為 null，則回傳 0，反之回傳 expression
var result = expression ?? 0

// 方法
public string SayHi(cName) => $"{cName}, Hi ~";

// 匿名型別
var o = new { OperaID = 1, Title = "Euridice", Year = 1600, Composer = "Jacopo Peri" };

// 陣列宣告
string[] Sizes = new[] { "Small", "Medium", "Large" }

public class Person {
	
	// get; set;
	public string name {get; set;} = "Lin";
	
	// get
	public int age => 42;
	
	public string phone {		
		get { return phone;}
		set { _phone = value;}		
	}
	
	public string addr {		
		get => "豐河社區"
		set => _addr = value;
	}	
}
```

```csharp=
@Html.Partial("_Links")    // ~/View/Shard/_Links.cshtml
@Html.Action()            // 直接執行 Action，return PartialView() 渲染頁面後，再顯示在本頁
@Html.ActionLink()        // 生成一個連結

return RedirectToAction()	// 讓 Browser 重發 request 

@Ajax.AcionLink()                // 產生Ajax連結
@using(Ajax.BeginForm()){}	// 產生Ajax表單

@Url.Action()	// 產生要執行 action 的路徑
@Url.Content()	// 產生要讀取靜態資源的路徑
```

=============================================

[Display(Name = "公告編號")]
[Required(ErrorMessage = "請輸入公告內容!")]
[MaxLength(100, ErrorMessage = "最多可輸入100字元")]
[DisplayFormat(DataFormatString = "{0:yyyy/MM/dd HH:mm}")]


@m				// 預設進行編碼
@Html.Raw(m)	// 不進行編碼

@Url.Content("~/js/TableSort.js?20220617182000")
@Html.ActionLink("首頁","Index")
html helper VS tag helper
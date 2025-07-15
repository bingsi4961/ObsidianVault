---
title: Json 序列化 / 反序列化
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5]

---

1. `[FromBody]`
自動將 HTTP 請求主體中的 JSON 數據反序列化為物件。

```csharp
public class CustomerController : ApiController 
{
    public IActionResult CreateCustomer([FromBody] Customer customer)
    {
        // customer 已經自動從 JSON 轉換為 Customer 物件
        _customerService.Add(customer);
        return Ok();
    }
}
```


2. `return Json(data)`

```csharp
public ActionResult GetCustomer()
{
    var customer = new Customer 
    { 
        Id = 1, 
        Name = "張小明"
    };
    
    return Json(customer); // HTTP 響應會自動設定 Content-Type: application/json
}
```

特性：
- 自動設定 Response 的 Content-Type 為 application/json
- 自動將物件序列化為 JSON 格式

3. `JsonSerializer.Deserialize<CustomerVM>(jsonData)`

```csharp
using System.Text.Json;

public void ProcessCustomerData(string jsonData)
{
    try 
    {
        // 將 JSON 字串反序列化為強型別物件。
        var customer = JsonSerializer.Deserialize<CustomerVM>(jsonData);
        Console.WriteLine($"客戶名稱: {customer.Name}");
    }
    catch (JsonException ex)
    {
        Console.WriteLine("JSON 解析錯誤: " + ex.Message);
    }
}
```

特性：
- .NET Core 內建的反序列化方法
- 效能較好
- 適合基本的反序列化需求

4. `return JsonSerializer.Serialize(data)`

```csharp
// 使用 System.Text.Json (.NET Core 內建)
using System.Text.Json;

public string SerializeCustomer()
{
    var customer = new Customer 
    { 
        Id = 1, 
        Name = "張小明",
        CreateDate = DateTime.Now
    };
    
    var options = new JsonSerializerOptions
    {
        WriteIndented = true,
        PropertyNamingPolicy = JsonNamingPolicy.CamelCase
    };
    
    return JsonSerializer.Serialize(customer, options);
}
```

特性：
- 效能較 Newtonsoft.Json 好
- 功能相對較少
- 適合簡單的序列化需求


5. `JsonConvert.DeserializeObject<CustomerVM>(jsonString)`

```csharp
// 使用 Newtonsoft.Json (Json.NET)
using Newtonsoft.Json;

public void ProcessJsonData(string jsonString)
{
    var settings = new JsonSerializerSettings
    {
        DateFormatHandling = DateFormatHandling.IsoDateFormat,
        NullValueHandling = NullValueHandling.Ignore
    };
    
    // 將 JSON 字串反序列化為強型別物件。
    var customer = JsonConvert.DeserializeObject<CustomerVM>(jsonString, settings);
}
```

特性：
- Newtonsoft.Json 的反序列化方法
- 可處理複雜的 JSON 結構
- 有豐富的自訂選項
- 適合需要特殊處理的反序列化場景

6. `return JsonConvert.SerializeObject(data)`

```csharp
// 使用 Newtonsoft.Json (Json.NET)
using Newtonsoft.Json;

public string SerializeWithJsonNet()
{
    var order = new Order
    {
        OrderId = "A001",
        Items = new List<OrderItem>
        {
            new OrderItem { ProductName = "筆記本", Quantity = 2 }
        }
    };
    
    return JsonConvert.SerializeObject(order, Formatting.Indented);
}
```

特性：
- 來自 Newtonsoft.Json 套件
- 較舊但功能完整的 JSON 處理函式庫
- 可自訂序列化設定
- 適合當需要取得 JSON 字串時使用

7. `return 物件 (直接回傳物件)`
在 ASP.NET Core Web API 中，直接返回物件會自動被序列化為 JSON。這是最簡潔的方式，框架會處理所有序列化細節。

```csharp
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpGet("{id}")]
    public Customer GetCustomer(int id)
    {
        var customer = new Customer 
        { 
            Id = id, 
            Name = "張小明" 
        };
        
        return customer; // ASP.NET Core 會自動序列化為 JSON
    }
}
```

特性：
- ASP.NET Core 會自動處理序列化
- 使用預設的序列化設定
- 程式碼最簡潔
- 適合簡單的 API 回傳場景

主要差異總結：
1. System.Text.Json (JsonSerializer) 是較新的實現，效能更好，但功能相對較少。
2. Newtonsoft.Json (JsonConvert) 功能更豐富，但效能較差。
3. ASP.NET Core 的自動序列化最為方便，適合大多數常見場景。
4. [FromBody] 特性用於接收客戶端發送的 JSON 數據。

使用建議：
- 對於新專案，優先使用 System.Text.Json
- 如果需要特殊的序列化功能，可以使用 Newtonsoft.Json
- 在 Web API 中，盡可能使用框架的自動序列化功能
- 需要細粒度控制時，才使用明確的序列化/反序列化方法

# 讓我用表格來整理不同的 JSON 處理方式：

1. **主要使用場景比較**

| 方法 | 使用場景 | 優點 | 缺點 |
|------|----------|------|------|
| `return Json(data)` | Controller 方法直接回傳給前端 | - 自動處理 Content-Type<br>- 框架內建方法<br>- 使用簡單 | - ==只能在 Controller 中使用==<br>- 較難自訂序列化設定 |
| `JsonConvert.SerializeObject()` | 需要將物件轉為 JSON 字串 | - 功能豐富<br>- 彈性高<br>- 可高度自訂 | - 需額外安裝套件<br>- 效能較差 |
| `JsonSerializer.Serialize()` | 簡單的序列化需求 | - .NET Core 內建<br>- 效能好<br>- 不需額外套件 | - 功能較少<br>- 彈性較低 |
| `return 物件` | 簡單的 API 回傳 | - 程式碼最簡潔<br>- 框架自動處理 | - 較難自訂序列化行為 |

2. **序列化和反序列化比較**

| 操作類型  | Newtonsoft.Json                      | System.Text.Json                  |
| ----- | ------------------------------------ | --------------------------------- |
| 序列化   | `JsonConvert.SerializeObject()`      | `JsonSerializer.Serialize()`      |
| 反序列化  | `JsonConvert.DeserializeObject<T>()` | `JsonSerializer.Deserialize<T>()` |
| 效能    | 較慢                                   | 較快                                |
| 功能豐富度 | 高                                    | 中                                 |
| 使用難度  | 簡單                                   | 中等                                |

3. **建議使用情境**

| 情境 | 建議方法 | 範例程式碼 |
|------|----------|------------|
| Controller API 回傳 | `return Json()` 或直接回傳物件 | `return Json(new { success = true, data });`<br>`return customerList;` |
| 需要 JSON 字串 | `JsonSerializer.Serialize()` | `var jsonString = JsonSerializer.Serialize(data);` |
| 複雜序列化需求 | `JsonConvert.SerializeObject()` | `var jsonString = JsonConvert.SerializeObject(data, settings);` |
| 基本反序列化 | `JsonSerializer.Deserialize<T>()` | `var obj = JsonSerializer.Deserialize<CustomerVM>(jsonString);` |
| 複雜反序列化 | `JsonConvert.DeserializeObject<T>()` | `var obj = JsonConvert.DeserializeObject<CustomerVM>(jsonString, settings);` |

4. **效能考量**

| 需求 | 建議選擇 | 原因 |
|------|----------|------|
| 高效能 | System.Text.Json | 專門針對 .NET Core 優化 |
| 功能完整 | Newtonsoft.Json | 提供更多序列化選項和功能 |
| 簡單使用 | 直接回傳物件 | 框架自動處理序列化 |
| 客製化需求 | Newtonsoft.Json | 提供最多自訂選項 |

# 哪些情況需要設定 Content-Type

1. **不需要手動設定 Content-Type 的情況**：
```csharp
// return Json() - 會自動設定 Content-Type: application/json
public ActionResult GetCustomer()
{
    var customer = new Customer { Id = 1, Name = "王大明" };
    return Json(customer);  // 自動設定
    // MVC5
    // return Json(customer, JsonRequestBehavior.AllowGet);
}

// .NET Core 直接回傳物件 - 會自動設定 Content-Type: application/json
public Customer GetCustomerObject()
{
    return new Customer { Id = 1, Name = "王大明" };  // 自動設定
}

// 使用 Ok() - 會自動設定 Content-Type: application/json
public IActionResult GetCustomerOk()
{
    var customer = new Customer { Id = 1, Name = "王大明" };
    return Ok(customer);  // 自動設定
}
```

2. **需要手動設定 Content-Type 的情況**：
```csharp
// 使用 JsonConvert.SerializeObject() 時需要設定
public ActionResult GetCustomerJsonConvert()
{
    var customer = new Customer { Id = 1, Name = "王大明" };
    var jsonResult = JsonConvert.SerializeObject(customer);
    
    // 需要設定，否則會被當作純文字處理
    // Response.Headers.Add("Content-Type", "application/json");
    Response.ContentType = "application/json";
    return Content(jsonResult);
}

// 使用 JsonSerializer.Serialize() 時需要設定
public ActionResult GetCustomerJsonSerializer()
{
    var customer = new Customer { Id = 1, Name = "王大明" };
    var jsonResult = JsonSerializer.Serialize(customer);
    
    // 需要設定，否則會被當作純文字處理
    Response.ContentType = "application/json";
    return Content(jsonResult);
    
    /*
    return Content(
        JsonSerializer.Serialize(customer),
        "application/json"
    );
    */
}
```

主要原則是：
- 使用框架提供的方法（如 `Json()`、`Ok()`）時，框架會自動處理 Content-Type
- 直接使用序列化方法（如 `JsonConvert.SerializeObject`、`JsonSerializer.Serialize`）時，因為只是產生 JSON 字串，需要手動設定 Content-Type
- 如果使用 `Content()` 方法回傳，也需要明確指定 Content-Type

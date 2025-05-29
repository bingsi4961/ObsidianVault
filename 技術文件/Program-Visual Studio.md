---
title: Program-Visual Studio
tags: [技術文件]

---

# Program-Visual Studio
###### tags: `技術文件`

---
## Singles
- [無法載入檔案或組件- 使用64 位元版本的IIS Express](https://blog.poychang.net/oracle-dataaccess/)
- [[C#.NET 拾遺補漏]10：理解volatile 關鍵字- 知乎](https://zhuanlan.zhihu.com/p/269031397)
- [ASP.NET 如何取得 Request URL 的各個部分 | The Will Will Web](https://blog.miniasp.com/post/2008/02/10/How-Do-I-Get-Paths-and-URL-fragments-from-the-HttpRequest-object))
- [[ASP.NET] 使用 acsx 與 aspx - WebUserControl @ 黃昏的甘蔗 :: 隨意窩 Xuite日誌](https://blog.xuite.net/tolarku/blog/48519600-%5BASP.NET%5D+%E4%BD%BF%E7%94%A8+acsx+%E8%88%87+aspx+-+WebUserControl)
- [[食譜好菜] 用 SqlBulkCopy 批次 Insert 大量資料讓你想不到的快](https://dotblogs.com.tw/supershowwei/2016/12/09/221622)
- [[食譜好菜] 用 SqlBulkCopy 可以快速批次 Insert 大量資料，那批次 Update 大量資料呢？](https://dotblogs.com.tw/supershowwei/2016/12/30/154257)
- [[C#]提高 Update、Delete 效能](https://dotblogs.com.tw/ricochen/2012/08/30/74396)
- [連線 Oracle Database 語法教學 - 理財工程師 Mars](https://blog.hungwin.com.tw/csharp-oracle-conn-syntax/)



---
## 目前無法叫用中斷點，未載入這個文件符號 
- [[Visual Studio] 偵錯模式無法叫用中斷點 | 史丹利好熱 - 點部落](https://dotblogs.com.tw/stanley14/2016/03/11/190123)
:::info
另外如果是網頁的部份，因為IIS Express 是用 32Bi t來跑，所以要選擇 X86 來執行
:::
![Uploading file..._k8t6rv80d](https://cko2xt-zoho.quip.com/-/blob/QaQAAA0Za5H/AeICVyk7KaFh27chdJYXoA)

---
## 釐清 CLR、.NET、C#、Visual Studio、ASP.NET 各版本之間的關係
- [釐清 CLR、.NET、C#、Visual Studio、ASP.NET 各版本之間的關係 | The Will Will Web](https://blog.miniasp.com/post/2015/07/28/Clarify-the-versions-between-CLR-NET-CSharp-Visual-Studio-and-ASPNET)
- [.NET Framework版本与CLR版本之间的关系 - mdgoogle - 博客园](https://www.cnblogs.com/atuo/p/6823788.html)
- [C# - .net framework和CLR各版本之间的关系_等等... 大雄，有啦！-CSDN博客](https://blog.csdn.net/gykimo/article/details/26971515)

---
## 頁面執行順序
- [ASP.NET頁面事件執行順序  | Boeis DreamFactory - 點部落](https://dotblogs.com.tw/boei/2010/07/10/16484)
- [Asp.NET頁面中事件載入的先後順序詳解 | 程式前沿](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/202025/)

---
## AutoResetEvent、ManualResetEvent
- [C# 深入理解AutoResetEvent和ManualResetEvent-阿里雲開發者社區](https://developer.aliyun.com/article/335066)
- [C# ManualResetEvent 与 AutoResetEvent 区别](https://blog.csdn.net/Pei_hua100/article/details/107105769)
- [C# 中ManualResetEvent用法總結WalkonNet](https://walkonnet.com/archives/13861)
- [C# ManualResetEvent用法](https://www.uj5u.com/net/119008.html)
- [C#-AutoResetEvent 與ManualResetEvent 控制執行緒暫停及恢復運作](https://hoohoo.top/blog/c-autoresetevent-and-manualresetevent-control-thread-suspend-and-resume-operation/)

::::success
- [什麼時候應該處理ManualResetEvent？- C# _程式人生](https://www.796t.com/post/OW9ob3E=.html)

:::info
- new AutoResetEvent(true)：預設訊號開啟
- new AutoResetEvent(false)：預設訊號關閉
- new ManualResetEvent(true)：預設訊號開啟
- new ManualResetEvent(false)：預設訊號關閉
:::

### AutoResetEvent
:::info
- WaitOne()：等待接收訊號
- Set()：開啟訊號，僅允許單個Thread通過(其他Thread等待下次Set())；WaitOne()執行後，自動關閉訊號
:::

### ManualResetEvent
:::info
- WaitOne()：等待接收訊號
- Set()：開啟訊號，允許所有Thread通過；WaitOne()執行後，訊號維持原狀態(保持通過)
- Reset()：關閉訊號
:::
::::

---
## InterLocked
- [C# 使用Interlocked實現線程同步- 台部落](https://www.twblogs.net/a/5bd74c202b71777ac86b4b3a)
- [C# 之執行緒同步淺析（1）-----輕量級同步Interlocked - IT閱讀](https://www.itread01.com/content/1550434510.html)
- [C# 多線程系列(3)：原子操作- 痴者工良- 博客園](https://www.cnblogs.com/whuanle/p/12724371.html)

---
## .Net Remoting
:::: success
### ★★★
:::info
- [C# .Net Remoting 基本說明(使用Interface 和組態檔) - LinBay](https://sites.google.com/site/willsnote/Home/remote)
- [C# 實現.Net Remoting服務端與客戶端通信|C/S框架網](http://www.csframework.com/archive/2/arc-2-20110626-1670.htm)
:::

### ★★
:::info
- [如何建立.NET Remoting 專案| 余小章@ 大內殿堂- 點部落](https://dotblogs.com.tw/yc421206/2020/06/15/how_to_create_remoting_object)
- [Microsoft .Net Remoting系列教程之一:.Net Remoting基礎篇| 程式前沿](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/208520/)
- [Remoting:什麼是Remoting，簡而言之，我們可以將其看作是一-百科知識中文網](https://www.easyatm.com.tw/wiki/Remoting)
:::

### ★
:::info
- [連接到.NET遠程服務器對象的最簡單方法是什麼](https://tw.coderbridge.com/questions/0cd52cab40c64f85b53fd97e29c46907)
:::
::::

---
## Thread Pool
- [C# 學習筆記執行緒集區與Execution Context - Huan-Lin 學習筆記](https://www.huanlintalk.com/2013/06/csharp-notes-multithreading-4-thread.html)
- [ThreadPool.QueueUserWorkItem多執行緒| 高級打字工!!! - 點部落](https://dotblogs.com.tw/ken74114/2010/12/08/19988)
- [C#中控制執行緒池的執行順序- IT閱讀](https://www.itread01.com/content/1545787637.html)
- [C# ThreadPool.QueueUserWorkItem方法代碼示例- 純淨天空](https://vimsky.com/zh-tw/examples/detail/csharp-method-system.threading.threadpool.queueuserworkitem.html)

---
## Thread.IsAlive
- ==如果這個執行緒已經啟動但還沒有正常終止或者中止，則為true ，否則為false。==
- [Thread.IsAlive 屬性(System.Threading) | Microsoft Docs](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.thread.isalive?view=net-5.0)

---
## WebRequest
- [如何使用 WebRequest,HttpWebRequest 來存取(GET,POST,PUT,DELETE,PATCH)網路資源 - Yo...](https://blog.yowko.com/webrequest-and-httpwebrequest/)
- [[ASP.NET] 使用 HttpWebRequest POST/GET 方法 (解決錯誤417,Server重新導向) | .Net 蛤什...](https://dotblogs.com.tw/joysdw12/2012/12/04/85380)
- [.NET 中如何選擇 WebClient，HttpClient，HttpWebRequest - CodingNote.cc](https://codingnote.cc/zh-tw/p/341688/)

---
## Web Service (.asmx)
- [[VS] C# web service 新增&使用 - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天](https://ithelp.ithome.com.tw/articles/10212151)
- [使用ASP.NET建立Web Service](https://www.cc.ntu.edu.tw/chinese/epaper/0029/20140620_2909.html)
- [[轉貼] ASP.NET -- Web Service (.asmx) & JSON | ASP.NET專題實務 WebForm  MVC教...](https://dotblogs.com.tw/mis2000lab/2015/09/18/web-service-responseformat-json)
- [[筆記] Web Service 概述 | m@rcus 學習筆記 - 點部落](https://dotblogs.com.tw/marcus116/2011/08/28/34524)
- [[.NET]分享使用Web Service介接的方式 | 亂馬客 - 點部落](https://dotblogs.com.tw/rainmaker/2013/08/29/115766)
- [[.NET]分享使用Web Service介接的方式(II) | 亂馬客 - 點部落](https://dotblogs.com.tw/rainmaker/2013/08/30/115913)

---
## HashTable
- [C# HashTable 雜湊表 & DictionaryEntry & foreach | chis coding life - 點部落](https://www.dotblogs.com.tw/chichiBlog/2018/10/16/172518)
- [C# Hashtable類 - C#教學](http://tw.gitbook.net/csharp/csharp_hashtable.html)
- [C# Hashtable類代碼示例 - 純淨天空](https://vimsky.com/zh-tw/examples/detail/csharp-class-system.collections.hashtable.html)
- [在C#.NET 1.1中打印出哈希表的鍵和數據 (Print out the keys and Data of a Hashtable in...](https://www.coderbridge.com/discussions/0001b8e7c15f4fd588a703609c1312cb)
- [C# 雜湊表（Hashtable）](https://twcode01.com/csharp/csharp-hashtable.html)
- [聊聊C# 中HashTable與Dictionary的區別說明 - IT145.com](https://www.it145.com/9/78261.html)
- [C# 3.0 初始設定 Hashtable 的方式 | The Will Will Web](https://blog.miniasp.com/post/2008/05/27/Initialize-a-Dictionary-with-a-Collection-Initializer)

---
## 委派 Delegate

```sql=
public delegate TResult Func<out TResult>();			// Func<TResult> 	=> 宣告 delegate TResult Func();
public delegate TResult Func<in T, out TResult>(T arg);	// Func<T,TResult> 	=> 宣告 delegate TResult Func(T arg1)
public delegate void Action();					// Action 		=> 宣告 delegate void Action();
public delegate void Action<in T>(T obj);			// Action<T> 		=> 宣告 delegate void Action(T arg1);
```

- [[Day8] 委派(delegate)，Action<T> ，與Func<T,TResult>](https://ithelp.ithome.com.tw/m/articles/10234953)
- [[C#]何謂委派(delegate)?. 委派:delegate是一個類別，在實體化委派事件時，可以自行決定這個委派事件要… | by Wi...](https://medium.com/@WilliamWhetstone/c-%E4%BD%95%E8%AC%82%E5%A7%94%E6%B4%BE-delegate-e7eec68da4e2)
- [(C#)委託delegate,Func<>,Action 解說系列(二)](https://isdaniel.github.io/c-func-2/)
- [C#雜記 — 委派(Delegate)、FUNC<>、ACTION<> | by 莊創偉 | Medium](https://ad57475747.medium.com/c-%E9%9B%9C%E8%A8%98-%E5%A7%94%E6%B4%BE-delegate-func-action-4b3191c7a131)
- [[C#] 委派 Delegate 與 Lambda | Jesper程式學習筆記 - 點部落](https://dotblogs.com.tw/JesperLai/2018/04/13/011011)
- [[C#] 委派 Delegate | 初探.NET的新手 - 點部落](https://dotblogs.com.tw/kevintan1983/2018/01/07/143620)

---
## Lambda
- [對C#的=>(Lambda)的理解. 感謝Eric大大給一個好理解的範例： | by 陳國仁 | Medium](https://medium.com/@newpage0720/%E5%B0%8Dc-%E7%9A%84-lambda-%E7%9A%84%E7%90%86%E8%A7%A3-438e6de01305)
- [C# 學習筆記 運算式 與 陳述式 Lambda | sqz777 der 技術小本本 - 點部落](https://dotblogs.com.tw/Im_sqz777/2017/07/23/225131)
- [C# λ運算子=>匿名方法 lambda表示式 - IT閱讀](https://www.itread01.com/content/1546123889.html)

---
## Log4net
- [关于Log4Net的使用及配置方式 - shanzm - 博客园](https://www.cnblogs.com/shanzhiming/p/12207896.html#JumpTo6)
- [關於Log4Net的使用及配置方式 - IT閱讀](https://www.itread01.com/content/1579330922.html)
- [log4net 中logger默认日志等级的设计意图详解 - 张志敏 - 博客园](https://www.cnblogs.com/beginor/archive/2009/08/28/1556029.html)
- [[ASP.NET] 使用 Log4net 記錄網站訊息 | .Net 蛤什麼? - 點部落](https://dotblogs.com.tw/joysdw12/2012/09/17/74860)
- [簡單記錄 log4net 的用法 | 學習之路 - 點部落](https://dotblogs.com.tw/yuanlin/2012/05/30/72479)
- [[.Net]快速設定寫log檔方法(log4net) | kevinya - 點部落](https://dotblogs.azurewebsites.net/kevinya/2017/03/27/113534)
- [[C#] 程式中 加入 log4net | 小盒子的星空 - 點部落](https://dotblogs.com.tw/Frank_Information_Workstation/2016/06/22/120008)

---
## URL 處理
- [ASP.NET 如何取得 Request URL 的各個部分 | The Will Will Web](https://blog.miniasp.com/post/2008/02/10/How-Do-I-Get-Paths-and-URL-fragments-from-the-HttpRequest-object)
- [在 C# 中獲取當前頁面的 URL | D棧 - Delft Stack](https://www.delftstack.com/zh-tw/howto/csharp/get-url-of-current-page-in-csharp/)
- [分享一個在 .NET / C# / ASP.NET 中修改網址結構的好方法 | The Will Will Web](https://blog.miniasp.com/post/2009/09/03/How-to-modify-Url-structure-in-aspnet-and-csharp)
- [CODEURL調整HTTP/HTTPSPortQueryString參數公用函式-黑暗執行緒](https://blog.darkthread.net/blog/uribuilder-enhancement/)
- [講解 URL 結構與分享幾個相對路徑與絕對路徑的開發技巧 | The Will Will Web](https://blog.miniasp.com/post/2008/10/19/URL-URI-Description-and-usage-tips)

---
## 日期處理
- [威力強大的 DateTime.ParseExact() | 瓶水相逢 - 艾小克 - 點部落](https://dotblogs.com.tw/chhuang/2008/03/18/1921)
- [TIPS-.NET DateTime Formating-黑暗執行緒](https://blog.darkthread.net/blog/tips-net-datetime-formating)

---
## Json
    
```csharp=
string jsonStr = Newtonsoft.Json.JsonConvert.SerializeObject(StuObject)
Student StuObj = Newtonsoft.Json.JsonConvert.DeserializeObject<Student>(jsonStr)
    
string jsonStr = System.Text.Json.JsonSerializer.Serialize<Student>(StuObject)    
Student StuObj = System.Text.Json.JsonSerializer.Deserialize<Student>(jsonStr)
```

- [[Day 22] C#中Json的序列化和反序列化的幾種方式(三)](https://ithelp.ithome.com.tw/articles/10195057)
- [用 C# 將 JSON 字串轉換為 Dictionary 物件 & 樹狀圖格式（JsonConvert.SerializeObject）](https://hackmd.io/@rpwis/SyktZcGvP)
- [C# Newtonsoft 讀寫 Json](https://hackmd.io/@EiTAESd2RbmdtpCg98qMvg/Syree6575)
- [[.NET] [C#] [JSON.NET] Serialize序列化與Deserialize反序列](https://dotblogs.com.tw/berrynote/2016/08/18/200338)
- [使用原生 System.Text.Json 命名空間處理 JSON](https://blog.poychang.net/using-json-serializer-with-system-text-json/)
    
---

JToken
├── JContainer
│   ├── JObject
│   ├── JArray
│   └── JProperty
└── JValue

* JArray
JavaScript陣列
含有多個JObject

* JObject
JavaScript物件
含有多個JProperty

* JProperty
JavaScript物件(JObject)之屬性，為Key-Value對應
JArray、JObject都可為JProperty的Value
    
    
- [[C#] JSON.Net 處理動態物件](https://www.pcbean.com/2018/06/c-jsonnet.html)
    
```csharp=
static void Main(string[] args)
{
    JObject john = new JObject();   // 建立物件
    john.Add(new JProperty("Name", "John"));                        // 加入屬性名稱及屬性值
    john.Add(new JProperty("Birthday", new DateTime(1993, 6, 10))); // 加入屬性名稱及屬性值

    JObject mary = new JObject();   // 建立物件
    mary.Add(new JProperty("Name", "John"));                        // 加入屬性名稱及屬性值
    mary.Add(new JProperty("Birthday", new DateTime(1993, 6, 10))); // 加入屬性名稱及屬性值

    JArray array = new JArray();    // 建立陣列
    array.Add(john);
    array.Add(mary);

    string json1 = JsonConvert.SerializeObject(array, Formatting.Indented); // 將JSON物件序列化成文字
    Console.WriteLine(json1);

    // [
    //   {
    //     "Name": "John",
    //     "Birthday": "1993-06-10T00:00:00"
    //   },
    //   {
    //     "Name": "John",
    //     "Birthday": "1993-06-10T00:00:00"
    //   }
    // ]

    Console.WriteLine("======================================");

    var ja = JArray.Parse(json1);
    JObject jo = (JObject)ja[0];
    Console.WriteLine($"Name:{jo?.Property("Name")?.Value}");
    Console.WriteLine($"Birthday:{jo?.Property("Birthday")?.Value}");

    JObject jo2 = (JObject)ja[1];
    foreach (var prop in jo2.Properties())
    {
        Console.WriteLine($"{prop.Name}:{prop.Value}");
    }

    // Name: John
    // Birthday:1993 / 6 / 10 上午 12:00:00
    // Name: John
    // Birthday:1993 / 6 / 10 上午 12:00:00


    Console.WriteLine("JObject 使用示例 ======================================");

    // 創建一個 JObject
    JObject person = new JObject(
        new JProperty("Name", "John Doe"),      // 加入屬性名稱及屬性值
        new JProperty("Age", 30),
        new JProperty("City", "New York")
    );

    // 訪問屬性
    string? name = (string?)person["Name"];
    int? age = (int?)person["Age"];

    // 添加新屬性
    person["Job"] = "Developer";

    // 序列化為 JSON 字符串
    string json = person.ToString();
    Console.WriteLine(json);

    //{
    //  "Name": "John Doe",
    //  "Age": 30,
    //  "City": "New York",
    //  "Job": "Developer"
    //}

    Console.WriteLine("JArray 使用示例 ======================================");

    // 創建一個 JArray
    JArray fruits = new JArray("Apple", "Banana", "Orange");

    // 添加元素
    fruits.Add("Grape");

    // 訪問元素
    string? firstFruit = (string?)fruits[0];

    // 遍歷數組
    foreach (var fruit in fruits)
    {
        Console.WriteLine(fruit);
    }

    Console.WriteLine("JProperty 使用示例 ======================================");

    // 創建一個 JProperty (物件內的屬性)
    JProperty property = new JProperty("Color", "Red");

    // 獲取屬性名和值
    string propertyName = property.Name;
    string? propertyValue = (string?)property.Value;

    // 將 JProperty 添加到 JObject
    JObject car = new JObject();
    car.Add(property);

    string json2 = JsonConvert.SerializeObject(car, Formatting.Indented);
    Console.WriteLine(json2);

    //{
    //    "Color": "Red"
    //}

    Console.WriteLine("JValue 使用示例 ======================================");

    // 創建不同類型的 JValue
    JValue stringValue = new JValue("Hello");
    JValue numberValue = new JValue(42);
    JValue boolValue = new JValue(true);

    // 獲取值
    string? str = (string?)stringValue;
    int num = (int)numberValue;
    bool boolean = (bool)boolValue;

    Console.WriteLine("解析 JSON 字符串 ======================================");

    string jsonString = @"{
        'Name': 'John Doe',
        'Age': 30,
        'Hobbies': ['Reading', 'Swimming']
    }";

    // 解析 JSON 字符串為 JObject
    JObject parsed = JObject.Parse(jsonString);

    // 訪問嵌套屬性
    string? name2 = (string?)parsed["Name"];
    JArray? hobbies = (JArray?)parsed["Hobbies"];

    // 遍歷 JArray
    foreach (var hobby in hobbies)
    {
        Console.WriteLine(hobby);
    }

    //Reading
    //Swimming

    Console.WriteLine("解析 JSON 字符串 ======================================");

    JObject data = JObject.Parse(@"{
        'Products': [
            { 'Name': 'Apple', 'Price': 0.5 },
            { 'Name': 'Banana', 'Price': 0.3 },
            { 'Name': 'Orange', 'Price': 0.4 }
        ]
    }");

    // 使用 LINQ 查詢 JSON
    var cheapProducts = data["Products"]
        .Where(p => (decimal)p["Price"] < 0.4m)
        .Select(p => (string)p["Name"]);

    foreach (var product in cheapProducts)
    {
        Console.WriteLine(product);
    }

    // Banana

    Console.WriteLine("JToken 使用示例 ======================================");
                
    string jsonString2 = @"{
        'Name': 'John Doe',
        'Age': 30,
        'IsStudent': false,
        'Courses': ['Math', 'Physics']
    }";

    // 解析 JSON 字符串為 JToken
    JToken token = JToken.Parse(jsonString2);

    // 檢查 token 的類型
    if (token is JObject)
    {
        Console.WriteLine("This is a JObject");
    }

    // 訪問屬性
    string name3 = (string)token["Name"];
    int age2 = (int)token["Age"];
    bool isStudent = (bool)token["IsStudent"];

    // 使用 SelectToken 方法訪問嵌套屬性
    string firstCourse = (string)token.SelectToken("Courses[0]");

    // 遍歷所有子元素
    foreach (var child in token.Children())
    {
        Console.WriteLine($"{child}");
    }

    // This is a JObject
    // "Name": "John Doe"
    // "Age": 30
    // "IsStudent": false
    // "Courses": [
    //   "Math",
    //   "Physics"
    // ]

    Console.WriteLine("JContainer 使用示例 ======================================");

    // 創建一個 JObject（JContainer 的子類）
    JObject person2 = new JObject(
        new JProperty("Name", "Jane Doe"),
        new JProperty("Age", 25)
    );

    // 使用 Add 方法添加新的屬性
    person2.Add("City", "London");

    // 使用 Remove 方法刪除屬性
    person2.Remove("Age");

    // 檢查是否包含某個屬性
    bool hasName = person2.ContainsKey("Name");

    // 獲取所有屬性名
    var propertyNames = person2.Properties().Select(p => p.Name);

    // 創建一個 JArray（也是 JContainer 的子類）
    JArray numbers = new JArray(1, 2, 3, 4, 5);

    // 使用 Add 方法添加新元素
    numbers.Add(6);

    // 使用 RemoveAt 方法刪除元素
    numbers.RemoveAt(0);

    // 獲取第一個和最後一個元素
    var first = numbers.First;
    var last = numbers.Last;

    // 遍歷 JContainer
    foreach (var item in person2)
    {
        Console.WriteLine($"{item.Key}: {item.Value}");
    }

    // Name: [Name, Jane Doe]
    // City: [City, London]

    Console.WriteLine("使用 JToken 和 JContainer 處理未知結構的 JSON ======================================");

    string complexJson = @"{
        'Name': 'John Doe',
        'Age': 30,
        'Address': {
            'Street': '123 Main St',
            'City': 'Anytown'
        },
        'PhoneNumbers': [
            '123-456-7890',
            '987-654-3210'
        ]
    }";

    JToken rootToken = JToken.Parse(complexJson);
    ProcessJson(rootToken);

    // .Name: John Doe
    // .Age: 30
    // .Address.Street: 123 Main St
    // .Address.City: Anytown
    // .PhoneNumbers[0]: 123 - 456 - 7890
    // .PhoneNumbers[1]: 987 - 654 - 3210
}

// 定義一個靜態方法，用於處理 JSON 結構
public static void ProcessJson(JToken token, string path = "")
{
    // 根據 JToken 的類型進行處理
    switch (token.Type)
    {
        // 當 JToken 類型為 Object（JSON 對象）時
        case JTokenType.Object:
            // 遍歷 JToken 的所有子節點，這些子節點都是 JProperty 物件
            foreach (var property in token.Children<JProperty>())
            {
                // 遞迴調用 ProcessJson 方法處理每個屬性值
                // 更新路徑為當前屬性名稱，並使用點（.）連接
                ProcessJson(property.Value, $"{path}.{property.Name}");
            }
            break;

        // 當 JToken 類型為 Array（JSON 陣列）時
        case JTokenType.Array:
            // 初始化索引值
            int index = 0;
            // 遍歷 JToken 的所有子節點，這些子節點是陣列元素
            foreach (var item in token.Children())
            {
                // 遞迴調用 ProcessJson 方法處理每個陣列元素
                // 更新路徑為當前索引位置，並使用方括號（[]）表示
                ProcessJson(item, $"{path}[{index}]");
                index++;
            }
            break;

        // 當 JToken 類型為基本數據類型（如字串、數字等）時
        default:
            // 輸出當前路徑和 JToken 的值
            Console.WriteLine($"{path}: {token}");
            break;
    }
}
```
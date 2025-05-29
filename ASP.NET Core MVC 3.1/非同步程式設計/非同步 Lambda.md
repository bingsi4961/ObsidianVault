---
title: 非同步 Lambda
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5]

---

## 非同步Lambda

只要在 Lambda 表示式前面加上 async 關鍵字，它就成了非同步匿名函式。

```csharp
static void Main(string[] args)
{
    // 此委派需傳入一個 string，且回傳一個 Task<string>。
    Func<string, Task<string>> downloadPageAsync =
        async (string url) => // 非同步 Lambda
        {
            using (var client = new HttpClient())
            {
                return await client.GetStringAsync(url);
            }
        };

     var task = downloadPageAsync("http://www.huan-lin.blogspot.com");
     Console.WriteLine(" 網頁⻑度: {0}", task.Result.Length);
 }
```

請注意，async Lambda 表示式和一般具名的 async 方法一樣，剛開始是同步執行的（亦即執行於呼叫端所在的執行緒），直到碰到第一個 await 敘述才會分出不同的控制流。
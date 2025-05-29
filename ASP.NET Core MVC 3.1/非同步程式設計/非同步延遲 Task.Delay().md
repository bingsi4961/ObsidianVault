---
title: 非同步延遲 Task.Delay()
tags: [ASP.NET Core MVC 3.1, ASP.NET MVC 5]

---

```csharp=
static async Task Main(string[] args)
{
    var task = MyTaskAsync();
    Console.WriteLine(" 已經返回 Main()");
    await task; // 等待工作完成
    Console.ReadLine();
}

static async Task MyTaskAsync()
{
    await Task.Delay(TimeSpan.FromSeconds(1)); // 非同步延遲 1 秒。
    Console.WriteLine(" 非同步工作結束"); // 延遲 1 秒後，回到這裡繼續執行。
}
```
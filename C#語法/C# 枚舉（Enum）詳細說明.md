---
title: 'C# 枚舉（Enum）詳細說明'
tags: ['C# 程式']

---

## 枚舉的基本結構

你的示例代碼定義了一個名為 `DeviceEnum` 的枚舉：

```csharp
public enum DeviceEnum
{
    [Description("單板")]
    PCBA = 8,
    [Description("整機")]
    Device = 11,
    紙板機 = 10,
}
```

### 1. 枚舉成員及其值

枚舉包含三個成員，每個都有一個關聯的整數值：
- `PCBA`：值為 8
- `Device`：值為 11
- `紙板機`：值為 10

==如果你不明確指定值，C# 會自動從 0 開始，並為每個後續成員加 1。==

### 2. Description 特性（Attribute）

`[Description("單板")]` 和 `[Description("整機")]` 是應用於枚舉成員的特性。這個特性來自 `System.ComponentModel` 命名空間，用於為枚舉值提供更多的描述性文本。要使用這個特性，你需要在代碼文件頂部添加：

```csharp
using System.ComponentModel;
```

## 枚舉的實際應用

### 基本使用方法

```csharp
DeviceEnum device = DeviceEnum.PCBA;
Console.WriteLine(device);         // 輸出: PCBA
Console.WriteLine((int)device);    // 輸出: 8
```

### 獲取 Description 特性值

要獲取 Description 特性中的描述文本，你需要使用反射：

```csharp
public static string GetDescription(Enum value)
{
    var field = value.GetType().GetField(value.ToString());
    var attribute = field.GetCustomAttribute<DescriptionAttribute>();
    return attribute == null ? value.ToString() : attribute.Description;
}

// 使用方式
DeviceEnum device = DeviceEnum.PCBA;
string description = GetDescription(device);  // 結果: "單板"
```

### 枚舉與字符串的轉換

```csharp
// 從字符串轉換為枚舉
string deviceName = "PCBA";
DeviceEnum device = (DeviceEnum)Enum.Parse(typeof(DeviceEnum), deviceName);

// 從枚舉轉換為字符串
string name = device.ToString();
```

### 遍歷所有枚舉值

```csharp
foreach (DeviceEnum device in Enum.GetValues(typeof(DeviceEnum)))
{
    Console.WriteLine($"名稱: {device}, 值: {(int)device}, 描述: {GetDescription(device)}");
}
```
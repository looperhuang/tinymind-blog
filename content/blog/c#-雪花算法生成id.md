---
title: C# 使用IdGen產生Id(雪花算法)
date: 2024-11-12T02:40:17.062Z
---

# 雪花算法

## 概述

雪花算法是 Twitter 開源的生成分散式全域產生 ID 的演算法

## 格式

雪花 ID 有 64 位元

- 1 位為符號位始終是 0
- 41 位為時間戳，精度毫秒
- 10 位可為(10 位機器 ID) 或 (5 位數據中心 ID + 5 位機器 ID)，可識別 1024 台機器
- 12 位為序列號 => 代表每毫秒可以有 4096 個 ID

![snowflake_structure.image](https://github.com/looperhuang/tinymind-blog/blob/main/assets/images/2024-11-12/1731380060528.png)

## 優缺點

### 優點

1. ID 包含時間戳，有序可排序
2. 簡單速度快

### 缺點

1. 依賴系統時間，如果系統時間回退或跳躍，會產生不唯一 ID
2. 自己手動分配機器 ID，分配不當，產生不唯一 ID

## Nuget
```
dotnet add package IdGen
```
## DI注入(可選)
```
dotnet add package IdGen.DependencyInjection
```

## Usage

### 一般
```C#
var generator = new IdGenerator(0);
var id = generator.CreateId();
```

### 依賴注入
Program.cs
```C#
builder.Services.AddIdGen(0);
```
Example.cs
```C#
public class Example
{
    //private readonly IIdGenerator<long> idGenerator;
    //public Example (IIdGenerator<long> idGenerator)
    //{
    //    this.idGenerator = idGenerator;
    //}
    
    private readonly IdGenerator idGenerator;
    public Example(IdGenerator idGenerator)
    {
        this.idGenerator = idGenerator;
    }

    public long GetId()
    {
        var id = idGenerator.CreateId();
        return id;
    }
}
```

## 參考資料
https://github.com/RobThree/IdGen?tab=readme-ov-file



---
title: C# 雪花算法生成ID
date: 2024-11-12T02:40:17.062Z
---

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



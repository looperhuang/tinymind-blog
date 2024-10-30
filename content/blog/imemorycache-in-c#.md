---
title: IMemoryCache In C#
date: 2024-10-30T01:29:22.602Z
---

## Nuget Package
Microsoft.Extensions.Caching.Memory
```shell
dotnet add package Microsoft.Extensions.Caching.Memory 
```

## Extensions.cs
```C#
public static void AddExtension(this IServiceCollection services)
{
    services.AddMemoryCache();
    // services.AddScoped<CacheHelper>(); //如果有使用整合工具類
}
```

## Example
```C#
 [Route("api/[controller]")]
 [ApiController]
 public class DataController : ControllerBase
 {
     private readonly DataBaseService dataBaseService;
     private readonly IMemoryCache memoryCache;

     public DataController(DataBaseService dataBaseService, IMemoryCache memoryCache)
     {
         this.memoryCache = memoryCache;
         this.dataBaseService = dataBaseService;
     }

     [HttpGet("/search")]
     public async Task<IActionResult> Search(string keyword)
     {
         if (memoryCache.TryGetValue("MainDataCache", out List<Prs006> dataList))
         {
             var result = dataList?
                 .Where(data => data.Ordnme?.Contains(keyword, StringComparison.OrdinalIgnoreCase) == true
                                || data.Scitn?.Contains(keyword, StringComparison.OrdinalIgnoreCase) == true
                                || data.Keyword?.Contains(keyword, StringComparison.OrdinalIgnoreCase) == true)
                 .ToList() ?? new List<Prs006>();

             return Ok(result);
         }
         else
         {
             var dbData = await dataBaseService.GetDatas();
             memoryCache.Set("MainDataCache", dbData, TimeSpan.FromDays(1));

             var result = dbData?
                 .Where(data => data.Ordnme?.Contains(keyword, StringComparison.OrdinalIgnoreCase) == true
                                || data.Scitn?.Contains(keyword, StringComparison.OrdinalIgnoreCase) == true
                                || data.Keyword?.Contains(keyword, StringComparison.OrdinalIgnoreCase) == true)
                 .ToList() ?? new List<Prs006>();

             return Ok(result);
         }
     }
```

## CacheHelper.cs(整合工具不一定需要)
```C#
public class CacheHelper
{
    private readonly IMemoryCache cache;

    public CacheHelper(IMemoryCache cache)
    {
        this.cache = cache;
    }

    public void Set<T>(string key, T value, int expire = 300)
    {
        var options = new MemoryCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(expire),
        };
        cache.Set(key, value, options);
    }

    public T? Get<T>(string key)
    {
        cache.TryGetValue<T>(key, out var value);
        return value;
    }

    public void Remove(string key)
    { 
        cache.Remove(key);
    }

    public void Update<T>(string key, T value, int expire)
    {
        cache.Remove(key);
        Set<T>(key, value, expire);
    }

    public bool Contains(string key)
    {
        return cache.TryGetValue(key, out _);
    }
}
```
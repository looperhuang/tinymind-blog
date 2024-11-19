---
title: C# Web Api 使用Quartz進行一次性排程
date: 2024-11-19T01:37:48.509Z
---

## Install
```powershell
Install-Package Quartz.AspNetCore
Install-Package Quartz.Extensions.DependencyInjection
Install-Package Quartz.Serialization.Json
```
## appsetting.json 
這邊使用Oracle資料庫做為持久化儲存[各類資料庫腳本](https://github.com/quartznet/quartznet/tree/main/database/tables)
```
"Quartz": {
  "quartz.scheduler.instanceName": "QuartzOracleScheduler",
  "quartz.scheduler.instanceId": "AUTO",
  "quartz.jobStore.type": "Quartz.Impl.AdoJobStore.JobStoreTX, Quartz",
  "quartz.jobStore.useProperties": "true",
  "quartz.jobStore.dataSource": "default",
  "quartz.jobStore.tablePrefix": "QRTZ_",
  "quartz.jobStore.driverDelegateType": "Quartz.Impl.AdoJobStore.OracleDelegate, Quartz",
  "quartz.dataSource.default.provider": "OracleODPManaged",
  "quartz.dataSource.default.connectionString": "Your Connection String"
}
```

## Job
```C#
 public class SendTasJob : IJob
 {
     public async Task Execute(IJobExecutionContext context)
     {
         //Job要實作的內容
          var tasid = context.MergedJobDataMap.GetString("tasid");
          var welcomeText = context.MergedJobDataMap.GetString("welcomeText");
          Console.WriteLine(tasid + welcomeText);
          return Task.CompletedTask;
     }
}
```

## Controller.cs
```C#
public class TasController : ControllerBase
{

    private readonly ISchedulerFactory schedulerFactory;
    private readonly IdGenerator idGenerator;

    public TasController(ISchedulerFactory schedulerFactory,
                         IdGenerator idGenerator)
    {
        this.schedulerFactory = schedulerFactory;
        this.idGenerator = idGenerator;
    }

    [HttpPost("[action]")]
    public async Task<IActionResult> ScheduleTasJob(TasJobRequest request)
    {
        // 使用 SchedulerFactory 取得 Scheduler
        var sch = await schedulerFactory.GetScheduler();
       // 雪花算法產Id
        var jobid = idGenerator.CreateId();
        var jobKey = new JobKey($"TasJob-{jobid}", "TasJobs");
        var triggerKey = new TriggerKey($"TasJobTrigger-{jobid}", "TasJobs");

        // 建立Job
        var job = JobBuilder.Create<SendTasJob>()
            .WithIdentity(jobKey)
             //key value 如果有開啟UseProperties 只能有string型別
            .UsingJobData("tasid", request.TasId)
            .UsingJobData("welcomeText", request.WelcomeText)
            .StoreDurably() // 使任務持久化
            .Build();

        // 設定Trigger
        ITrigger trigger = TriggerBuilder.Create()
            .WithIdentity(triggerKey)
            .StartAt(request.ExecuteAt ?? DateTime.UtcNow) // 指定執行時間
            .WithSimpleSchedule(x => x
                .WithMisfireHandlingInstructionFireNow() // 當排程錯過執行時間時立即執行
                .WithRepeatCount(0) //只執行一次
                )
            .Build();

        try
        {
           await sch.ScheduleJob(job, trigger);
           Console.WriteLine($"Scheduled job successfully with JobKey: {jobKey.Name} and TriggerKey: {triggerKey.Name}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Failed to schedule job with JobKey: {jobKey.Name} and TriggerKey: {triggerKey.Name}", ex);
            return StatusCode(500, "Failed to schedule job");
        }

        return Ok($"Scheduled job successfully with Job: {jobKey.Name} and Trigger: {triggerKey.Name}");
    }
}
```

## Program.cs
```C#
// 從 appsettings.json 載入 Quartz 配置
builder.Services.Configure<QuartzOptions>(builder.Configuration.GetSection("Quartz"));

// 加入 Quartz 服務
builder.Services.AddQuartz(q => 
{
    q.UseJobFactory<MicrosoftDependencyInjectionJobFactory>();

    // 持久化
    q.UsePersistentStore(opt =>
    {
        opt.UseProperties = false;
        opt.UseOracle(configuration =>
        {
            configuration.ConnectionString = builder.Configuration["Quartz:quartz.dataSource.default.connectionString"];
        });
        opt.UseNewtonsoftJsonSerializer();
    });
});
builder.Services.AddQuartzServer(q => q.WaitForJobsToComplete = true); //hosting server
//builder.Services.AddQuartzHostedService(options =>
//{
//    // when shutting down we want jobs to complete gracefully
//    options.WaitForJobsToComplete = true;
//});
builder.Services.AddTransient<SendTasJob>(); // Job註冊

```

## AddQuartzServer vs AddQuartzHostedService
AddQuartzServer 會多一個Health Check 檢查 scheduler   
Source Code如下
```C# 
namespace Quartz
{
    public static class QuartzServiceCollectionExtensions
    {
        public static IServiceCollection AddQuartzServer(
            this IServiceCollection services,
            Action<QuartzHostedServiceOptions>? configure = null)
        {
            #if SUPPORTS_HEALTH_CHECKS
            services
                .AddHealthChecks()
                .AddTypeActivatedCheck<QuartzHealthCheck>("quartz-scheduler");
            #endif

            return services.AddQuartzHostedService(configure);
        }
    }
}
```

## IIS回收機制，排程失效
- 關閉回收機制
![InetMgr_ndjANFedAK.png](https://github.com/looperhuang/tinymind-blog/blob/main/assets/images/2024-11-19/1731979706293.png?raw=true)
![InetMgr_8EXFtPapjN.png](https://github.com/looperhuang/tinymind-blog/blob/main/assets/images/2024-11-19/1731979731228.png?raw=true)
- 定期呼叫api維持server keep alive (ex.使用window排程)
```powershell
# File: InvokeApi.ps1

# 設定 URL
$url = "your api url"
# 設定 Log Path
$logFile = "C:\Scripts\InvokeApi.log"

# 調用 API
try {
    $response = Invoke-WebRequest -Uri $url -Method Get -TimeoutSec 10
    $logEntry = "$(Get-Date) - API call successfully: $($response.StatusCode)"
} catch {
    $logEntry = "$(Get-Date) - API call failed: $($_.Exception.Message)"
}

# 寫入日誌
$logEntry | Out-File -FilePath $logFile -Append

```
- 回收後，應用關閉事件添加應用重啟
![35fcb716e101ade84bd9ea48969cf1fe.png](https://github.com/looperhuang/tinymind-blog/blob/main/assets/images/2024-11-19/1731979950617.png?raw=true)

## 參考資料
https://www.quartz-scheduler.net/  
https://stackoverflow.com/questions/75443765/quartz-aspnetcore-and-hosting-in-asp-net-core  
https://blog.csdn.net/qq_18932003/article/details/139672249
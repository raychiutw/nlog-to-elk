# nlog-to-elk

## 事前安裝

1. Linux: 安裝 docker 與 docker-compose
2. Windows: 安裝 dokcer destop

## 安裝 ELK Container

下載 elk [docker-compose.yml](docker-compose.yml), 並切換到該目錄下



> 啟動 elk

```
docker-compose up -d
```

> 停用 elk

```
docker-compose 
```

## .Net 程式加入 nlog

> 安裝套件

```
NLog.Web.AspNetCore
NLog.Targets.ElasticSearch
```

> nlog.config

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      throwExceptions="false"
      autoReload="true">

	<!-- enable asp.net core layout renderers -->
	<extensions>
		<add assembly="NLog.Web.AspNetCore" />
		<add assembly="NLog.Targets.ElasticSearch" />
	</extensions>

	<targets async="true">
		<target name="elastic" xsi:type="BufferingWrapper" flushTimeout="5000">
			<target xsi:type="ElasticSearch"
				uri="http://localhost:9200/"
				index="pmp-nlog-elk-${date:format=yyyy.MM.dd}"
				documentType="logevent"
				includeAllProperties="false">
				<field name="message" layout="${message}" />
				<field name="exception" layout="${exception:format=toString}" />
				<field name="machineName" layout="${machinename}" />
				<field name="clientIp" layout="${aspnet-request-ip}" />
				<field name="traceId" layout="${aspnet-TraceIdentifier}" />
				<field name="time" layout="${longdate}" />
				<field name="level" layout="${level:uppercase=true}" />
				<field name="logger" layout=" ${logger}" />
				<field name="processId" layout=" ${processid}" />
				<field name="threadName" layout=" ${threadname}" />
				<field name="stackTrace" layout=" ${stacktrace}" />
			</target>
		</target>
	</targets>

	<rules>
		<logger name="*" minlevel="Info" writeTo="elastic" />
	</rules>
</nlog>
```

> Program.cs

修改 Main 與 CreateHostBuilder

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var logger = NLogBuilder.ConfigureNLog("nlog.config").GetCurrentClassLogger();
            try
            {
                logger.Debug("init main");
                CreateHostBuilder(args).Build().Run();
            }
            catch (Exception exception)
            {
                logger.Error(exception, "Stopped program because of exception");
                throw;
            }
            finally
            {
                NLog.LogManager.Shutdown();
            }
        }

        public static IHostBuilder CreateHostBuilder(string[] args)
        {
            return Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                .ConfigureLogging(logging =>
                {
                    logging.SetMinimumLevel(LogLevel.Trace);
                })
                .UseNLog();
        }
    }
```

## 連線管理工具

 [kaizen](https://www.elastic-kaizen.com/)

---
title: 从零开始搭建Maui框架04-添加日志组件
date: 2024-11-19 14:19:32
categories:
	- 教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---

## 日志系统

日志系统是应用程序中一个重要的组件，它负责记录应用程序在运行过程中的各种事件和信息。这些信息包括但不限于：

* **错误信息：** 程序运行时发生的异常、错误等。
* **调试信息：** 开发者为了调试程序而添加的详细输出。
* **警告信息：** 程序运行过程中出现的一些潜在问题或异常情况。
* **重要事件：** 程序中一些关键操作的记录，如用户登录、数据更新等。

### 日志系统的作用

* **故障排查：** 通过分析日志，可以快速定位并解决程序中的问题，提高系统的稳定性。
* **性能优化：** 分析日志可以发现性能瓶颈，优化程序的运行效率。
* **安全审计：** 记录用户操作和系统事件，有助于进行安全审计和风险评估。
* **数据分析：** 日志数据可以用于分析用户行为、系统使用情况等，为产品改进提供数据支持。

### 日志系统的组成

一个完整的日志系统通常包括以下几个部分：

* **日志生成：** 应用程序在关键位置生成日志信息，通常通过调用日志库的接口来实现。
* **日志收集：** 将分散在各个服务器上的日志收集到一个中心化的存储系统中。
* **日志存储：** 将收集到的日志数据存储在磁盘、数据库或云存储等介质上。
* **日志分析：** 对存储的日志数据进行分析，提取有价值的信息。

### 日志系统的分类

* **按功能分类：**
  * **错误日志：** 记录程序错误和异常信息。
  * **调试日志：** 记录调试信息，方便开发人员定位问题。
  * **访问日志：** 记录用户访问系统的信息，如访问时间、IP地址等。
  * **系统日志：** 记录系统运行状态和事件信息。
* **按技术分类：**
  * **文件日志：** 将日志信息写入到文本文件中。
  * **数据库日志：** 将日志信息存储到数据库中。
  * **消息队列日志：** 将日志信息发送到消息队列中。
  * **结构化日志：** 将日志信息以结构化的格式存储，方便搜索和分析。

## 在Maui中添加日志组件

支持MAUI的日志组件有很多，在这里，我们将使用`Serilog` 可以[点击这里](https://serilog.net/)查看官网。

### 添加Nuget包

我们需要添加`Serilog`、`Serilog.Sinks.Async` 、`Serilog.Sinks.File`、`Serilog.Extensions.Hosting`

### 配置日志组件

参考[官网文档](https://https://serilog.net/)，我们可以发现，只要配置好组件以后，就可以直接使用了。

我们在程序的入口处配置组件，代码如下

```
           prism.ConfigureLogging(configureLogging =>
                {
                    var logConfig = new LoggerConfiguration();

                    logConfig.WriteTo.Async((skinConfig) =>
                    {
                        var basePath = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);

#if ANDROID
                        var androidPaths = Android.App.Application.Context.GetExternalFilesDirs(null);
                        basePath = androidPaths[0].AbsolutePath;
#endif

                        var baseLogPath = Path.Combine(basePath, "logs");

                        skinConfig.File(path: $"{baseLogPath}/.log",
                            restrictedToMinimumLevel: LogEventLevel.Verbose,
                            rollingInterval: RollingInterval.Day);
                    });

                    Log.Logger = logConfig.CreateLogger();

                    configureLogging.Services.AddSerilog(Log.Logger);
                });
```

其中，basePath是日志文件存放的路径。当在Windows下时，直接获取`LocalApplicationData`路径。

**在Android中，我们将日志文件放置在外部存储中。**

> 在android中，我们需要添加外部文件读取权限`android.permission.READ_EXTERNAL_STORAGE`和外部文件写入权限`android.permission.WRITE_EXTERNAL_STORAGE`.

完整的注册文件如下

```
using Core;
using Microsoft.Extensions.Logging;
using Serilog;
using Serilog.Events;
using System.Diagnostics;
using UI;


namespace MauiPrism9Demo
{
    public static class MauiProgram
    {
        public static MauiApp CreateMauiApp()
        {
            var builder = MauiApp.CreateBuilder();
            builder.UseMauiApp<App>();


            builder.ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            });


#if DEBUG
            //启用日志调试
            Serilog.Debugging.SelfLog.Enable(Console.Error);
#endif


            builder.UsePrism(prism =>
            {
                prism.RegisterTypes(container => { container.RegisterForNavigation<MainPage>(); });


                prism.ConfigureLogging(configureLogging =>
                {
                    var logConfig = new LoggerConfiguration();

                    logConfig.WriteTo.Async((skinConfig) =>
                    {
                        var basePath = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);

#if ANDROID
                        var androidPaths = Android.App.Application.Context.GetExternalFilesDirs(null);
                        basePath = androidPaths[0].AbsolutePath;
#endif

                        var baseLogPath = Path.Combine(basePath, "logs");

                        skinConfig.File(path: $"{baseLogPath}/.log",
                            restrictedToMinimumLevel: LogEventLevel.Verbose,
                            rollingInterval: RollingInterval.Day);
                    });

                    Log.Logger = logConfig.CreateLogger();

                    configureLogging.Services.AddSerilog(Log.Logger);
                });


                prism.ConfigureModuleCatalog(configureCatalog =>
                {
                    configureCatalog.AddModule<CoreModule>(InitializationMode.OnDemand);
                    configureCatalog.AddModule<UiModule>();
                });


                //导航到根目录
                prism.CreateWindow($"/{nameof(MainPage)}", ex =>
                {
                    Debug.Write(ex.StackTrace);
                    throw ex;
                });
            });


#if DEBUG
            builder.Logging.AddDebug();
#endif

            return builder.Build();
        }
    }
}
```

## 使用日志组件

在任意需要使用日志的地方，直接调用`Log.Logger.xxx`就能够生成日志了。如在MainPage初始化的完成的时候，直接调用`Log.Logger.Information("MainPage初始化");`会在日志中写入`MainPage`初始化完成。

也可以通过依赖注入的方式，直接注入`ILogger`接口，然后直接调用`logger.xxx`即可。如在ViweAViewModel中调用

```
using System.Windows.Input;
using Core;
using Core.Dtos;
using Core.Events;
using Serilog;
using UI.Views;

namespace UI.ViewModels
{
    public class ViewAViewModel(IRegionManager regionManager, IEventAggregator eventAggregator, ILogger logger)
        : BindableBase
    {
        public ICommand ToViewBCommand => new DelegateCommand(() =>
        {
            regionManager.RequestNavigate(RegionNames.MainRegion, nameof(ViewB));
        });

        public ICommand PublishEventCommand => new DelegateCommand(() =>
        {
            eventAggregator.GetEvent<EventTest>().Publish(new EventTestDto("这是一个Title",
                $"这是一个Content，发布时间为：{DateTime.Now:yyy-MM-dd HH:mm:ss}"));

            logger.Information("发布消息");
        });
    }
}
```

## 总结

日志组件的使用无非就是配置，然后直接调用，非常简单。

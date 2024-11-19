---
title: 从零开始搭建Maui框架01-搭建基础框架
date: 2024-11-19 14:12:32
categories:
	- 教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---
# 从零搭建Maui框架，基于Prism9.0

Prism9.0正式版发布了，利用Prism搭建一个Maui框架。

## 一、用VS模板创建Maui项目

使用Visual Studio 2022及以上，创建.NET MAUI应用

![1724825693606](images/从零开始搭建Maui框架/1724825693606.png)

## 二、 导包

导入`Prism.Maui` 和`Prism.DryIoc.Maui` 两个Nuget包

![1724825979147](images/从零开始搭建Maui框架/1724825979147.png)

## 三、修改Maui程序入口

在`MauiProgram.cs` 中`CreateMauiApp` 方法中添加以下代码

```
builder.UsePrism(prism =>
{
    //注册MainPage到导航
    prism.RegisterTypes(container => { container.RegisterForNavigation<MainPage>(); });
   
    //导航到根目录
    prism.CreateWindow(navigationService => navigationService.NavigateAsync($"/{nameof(MainPage)}"));
});
```

上面代码的作用是，注册`MainPage`到Prism的导航服务中。如果不注册，直接使用`prism.CreateWindow`方法会报错，**提示创建Root页面失败**。

## 四、删除AppShell

使用Prism不需要`AppShell` ，这里直接删除。

在`App.xaml`文件中删除 `MainPage = new AppShell();` 代码。

直接运行程序，会发现程序的行为和VS模板创建的动作行为基本是一致的。

这里因为我们删除了AppShell，所以看不到Home字样。

## 五、添加区域`Region`

1. 将`MainPage.xaml`中的UI代码删除，新增`ContentView`控件
2. 导入命名空间`xmlns:prism="http://prismlibrary.com"`
3. 给`ContentView`附加区域导航属性`prism:RegionManager.RegionName="MainRegion"`
4. 注释`MainPage.xaml.cs`中报错的部分

`MainPage.xaml` 完整代码如下

```
<ContentPage x:Class="MauiPrism9Demo.MainPage"
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:core="clr-namespace:Core;assembly=Core"
             xmlns:prism="http://prismlibrary.com">

    <ContentView prism:RegionManager.RegionName="MainRegion" />


</ContentPage>

```

## 六、添加新视图

新增Views文件夹和ViewModels文件夹。

在Views文件夹中新增一个名为`ViewA`的`.NET MAUI ContentView(XAML)`项目

在ViewModels中新增名为`ViewAViewModel` 的类，并使**类继承Prism的MVVM基类`BindableBase`**

![1724828119533](images/从零开始搭建Maui框架/1724828119533.png)

## 七、注入新视图并导航

在**程序入口**处使用Prism**基于区域注册**新的视图。

代码为`container.RegisterForRegionNavigation<ViewA, ViewAViewModel>();`

新增Prism加载完成后导航到ViewA的代码。

```
  prism.OnInitialized((container) =>
  {
      var regionManager = container.Resolve<IRegionManager>();
      regionManager.RegisterViewWithRegion("MainRegion", "ViewA");
  });
```

上面这段代码是**在Prism初始化完成以后**，使用区域管理器`RegionManager` 将`ViewA`导航到`MainRegion`中。

运行程序，会发现ViewA的UI已经显示在了应用程序中。

## 八、区域导航

重复第六条，新增ViewB和ViewBViewModel。

在ViewB中添加一个按钮，绑定命令为ToViewACommand。

```
 <Button Command="{Binding ToViewACommand}"
         HorizontalOptions="Center"
         Text="GoBack"
         VerticalOptions="Center" />
```

在ViewBViewModel中新增一个返回值为`ICommand`的方法，代码如下

```
 public ICommand ToViewACommand => new DelegateCommand(() =>
 {
     regionManager.RequestNavigate("MainRegion", nameof(ViewA));
 });
```

**同样在ViewA和ViewAViewModel中新增ToViewB的按钮和Command。**

## 九、注册区域导航

在程序入口文件`MauiProgram` 中，像注册ViewA一样，新增ViewB和ViewBViewModel的注册，基于区域导航，

代码为`container.RegisterForRegionNavigation<ViewB, ViewBViewModel>();`

运行程序，ViewA和ViewB正常互相导航，基本框架完成。

## 十、完整代码

`MauiProgram.cs`

```
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

            builder.UsePrism(prism =>
            {
                prism.RegisterTypes(container =>
                {
                    container.RegisterForNavigation<MainPage>();
                    container.RegisterForRegionNavigation<ViewA, ViewAViewModel>();
                    container.RegisterForRegionNavigation<ViewB, ViewBViewModel>();
                });
                prism.OnInitialized((container) =>
                {
                    var regionManager = container.Resolve<IRegionManager>();
                    regionManager.RegisterViewWithRegion("MainRegion", nameof(ViewA));
                });
  
                //导航到根目录
                prism.CreateWindow(navigationService => navigationService.NavigateAsync($"/{nameof(MainPage)}"));
            });


#if DEBUG
            builder.Logging.AddDebug();
#endif

            return builder.Build();
        }
    }
}
```

`ViewA`

```
<?xml version="1.0" encoding="utf-8" ?>
<ContentView x:Class="MauiPrism9Demo.Views.ViewA"
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">
    <VerticalStackLayout HorizontalOptions="Center" VerticalOptions="Center">
        <Label HorizontalOptions="Center"
               Text="ViewA"
               VerticalOptions="Center" />
        <Button Command="{Binding ToViewBCommand}"
                HorizontalOptions="Center"
                Text="ToViewBCommand"
                VerticalOptions="Center" />
    </VerticalStackLayout>
</ContentView>

```

`ViewAViewModel`

```
namespace MauiPrism9Demo.ViewModels
{
    public class ViewAViewModel(IRegionManager regionManager) : BindableBase
    {
        public ICommand ToViewBCommand => new DelegateCommand(() =>
        {
            regionManager.RequestNavigate("MainRegion", nameof(ViewB));
        });
    }
}
```

## 十一、模块化

Prism.Maui的模块化和Prism.Wpf的模块化几乎是一样的。只需要在程序入口(`MauiProgram`)处注册模块即可。

```
     prism.ConfigureModuleCatalog(configureCatalog =>
     {
         configureCatalog.AddModule<CoreModule>();
         configureCatalog.AddModule<UiModule>();
     });
```

上面的代码添加了2个模块到应用程序中。模块只需要实现`IModule`接口即可。

我这里把ViewA、ViewB包括ViewModel都移动到了UiModule中，注册视图(ViewA、ViewB)的代码都移动到了UiModule中。CoreModule是一些业务代码。

Ui移动到别的模块中以后，`MauiProgram`中就可以将视图的注册和程序初始化导航都移除了。最终的`MauiProgram`代码为

```
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

            builder.UsePrism(prism =>
            {
                prism.RegisterTypes(container => { container.RegisterForNavigation<MainPage>(); });

                prism.ConfigureModuleCatalog(configureCatalog =>
                {
                    configureCatalog.AddModule<CoreModule>();
                    configureCatalog.AddModule<UiModule>();
                });

                //导航到根目录
                prism.CreateWindow(navigationService => navigationService.NavigateAsync($"/{nameof(MainPage)}"));
            });


#if DEBUG
            builder.Logging.AddDebug();
#endif

            return builder.Build();
        }
    }
}
```

自此，整个基础框架就搭建完成了。

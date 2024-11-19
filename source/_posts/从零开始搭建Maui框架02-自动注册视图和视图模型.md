---
title: 从零开始搭建Maui框架02-自动注册视图和视图模型
date: 2024-11-19 14:15:32
categories:
	- MAUI教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---
# 从零开始搭建Maui框架02-自动注册视图和视图模型

上一次我们创建了基础的Maui框架，现在我们进一步优化上次的框架。

我们在创建视图(`View`)和视图模型(`ViewModel`)后，需要再在模块(`Module`)中将这两者注册到容器中，比较麻烦，而且代码量也很大，还容易遗漏注册，最主要的是不够优雅，下面我们将实现视图和视图模型自动注册。

这个教程中用到的技术原理是反射(`reflection`)和特性(`Attribute`)以及扩展方法。


## 一、新增类库项目

为了后续更好的模块化，现在在解决方案中新增两个新的**Maui类库项目**，**分别命名为Core和MauiLib**

现在项目中有3个类库项目(`Core、MauiLib、UI`)，1个Maui应用程序项目(`MauiPrism9Demo`)。

Core：存放项目基础框架代码，这次新增的主要代码都在其中。

MauiLib：存放项目中用到的第三方类库，像之前`Prism.Maui、Prism.Ioc.Maui` 等都放在这里，**其他项目只需要引用这个项目，不需要重复导入第三方类库，同时也方便管理。**

MauiPrism9Demo：应用程序入口，模块加载，应用程序配置都在这里。

UI：主要存放视图和视图模型。后面可以把主题也放这里。

**应用程序的引用路径为：`Maui -> Core -> UI-> MauiPrism9Demo`，不要重复引用，以免造成循环引用。**

## 二、新增特性`Attribute`

在Core类库中新增Attributes文件夹，并在其中新增一个类，命名为`IocForRegionNavigationAttribute` ，继承`Attribute`

1.在类中新增两个自动属性

```
 /// <summary>
 /// ViewModel类型
 /// </summary>
 public Type ViewModelType { get; set; } = viewModelType;

 /// <summary>
 /// 注册名称
 /// </summary>
 public string? RegisterName { get; set; } = registerName;
```

`ViewModelType` 是用来表示要注入到视图的`ViewModel`类型

`RegisterName`是注册的名字，区域导航的时候用到，如果为空则使用`View`的`name`

2.添加构造函数，将两个自动属性通过构造赋值。我这里使用了C#12中的主构函数。

```
 public class IocForRegionNavigationAttribute(Type viewModelType, string? registerName = null) : Attribute
```

3.完整的代码

```
namespace Core.Attributes
{
    /// <summary>
    /// 注入视图模型到区域导航
    /// </summary>
    /// <param name="viewModelType"></param>
    /// <param name="registerName"></param>
    [AttributeUsage(AttributeTargets.Class)]
    public class IocForRegionNavigationAttribute(Type viewModelType, string? registerName = null) : Attribute
    {
        /// <summary>
        /// ViewModel类型
        /// </summary>
        public Type ViewModelType { get; set; } = viewModelType;

        /// <summary>
        /// 注册名称
        /// </summary>
        public string? RegisterName { get; set; } = registerName;
    }
}
```

代码中的`AttributeUsage`特性是指示当前这个特性(`IocForRegionNavigationAttribute`)只能用在类上。

## 三、使用特性

[特性官方文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/advanced-topics/reflection-and-attributes/)

在UI类库中，找到我们的视图(`View`)，在视图的隐藏代码上添加`IocForRegionNavigation`特性，代码如下

```
using Core.Attributes;
using UI.ViewModels;

namespace UI.Views;

[IocForRegionNavigation(typeof(ViewAViewModel))]
public partial class ViewA : ContentView
{
    public ViewA()
    {
        InitializeComponent();
    }
}
```

这里我们将`ViewAViewModel`作为`ViewA`的`ViewModel`

## 四、编写容器扩展

[扩展方法的官方文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)

在`Core`类库中新增`Extensions`文件夹，用来存放扩展方法。

新增静态类，命名`ContainerExtension`

类中新增`RegisterAllViewModel`扩展方法，代码如下

```
using Core.Attributes;
using System.Reflection;

namespace Core.Extensions
{
    public static class ContainerExtension
    {
        /// <summary>
        /// 自动注册所有的View和ViewModel
        /// </summary>
        /// <param name="container"></param>
        public static void RegisterAllViewModel(this IContainerRegistry container, Assembly assembly)
        {
            var types = assembly.GetTypes().Where(c => c.BaseType == typeof(ContentView));

            foreach (var type in types)
            {
                var attribute = type.GetCustomAttribute<IocForRegionNavigationAttribute>();

                if (attribute != null)
                {
                    var viewModelType = attribute.ViewModelType;
                    var registerName = attribute.RegisterName ?? type.Name;

                    container.RegisterForRegionNavigation(type, viewModelType, registerName);
                }
            }
        }
    }
}
```

`var types = assembly.GetTypes().Where(c => c.BaseType == typeof(ContentView));`是为了获取程序集中所有的`ContentView`，因为我们的特性都是放在`ContentView`上的，它承载了应用的UI。

我们这里找到所有使用了`IocForRegionNavigationAttribute`特性的`ContentView`，并根据`ViewModelType`注册进容器。

## 五、使用扩展方法

在UI类库中，修改UIModul

```
using Core;
using Core.Extensions;
using System.Reflection;
using UI.Views;

namespace UI
{
    public class UiModule(IRegionManager regionManager) : IModule
    {
        #region Implementation of IModule

        /// <summary>
        /// Used to register types with the container that will be used by your application.
        /// </summary>
        public void RegisterTypes(IContainerRegistry container)
        {
            //container.RegisterForRegionNavigation<ViewA, ViewAViewModel>();
            //container.RegisterForRegionNavigation<ViewB, ViewBViewModel>();

            container.RegisterAllViewModel(Assembly.GetAssembly(typeof(UiModule))!);
        }

        /// <summary>Notifies the module that it has been initialized.</summary>
        public void OnInitialized(IContainerProvider containerProvider)
        {
            regionManager.RegisterViewWithRegion(RegionNames.MainRegion, nameof(ViewA));
        }

        #endregion
    }
}
```

在这里我们注释掉了原来的视图注册方法，直接调用`container.RegisterAllViewModel(Assembly.GetAssembly(typeof(UiModule))!);`进行注册。

其他代码不需要改动。

自此，所有新建在Ui类库中的视图，只要使用了`IocForRegionNavigationAttribute` ,程序都会自动在加载的时候将视图注入到容器中。

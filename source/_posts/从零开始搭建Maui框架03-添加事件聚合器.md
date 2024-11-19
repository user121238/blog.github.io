---
title: 从零开始搭建Maui框架03-添加事件聚合器
date: 2024-11-19 14:18:32
categories:
	- 教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---
# 从零开始搭建Maui框架03-添加事件聚合器

## 一、事件聚合器：实现组件间松耦合通信的利器

**事件聚合器**（Event Aggregator）是一种设计模式，它提供了一种在应用程序中不同组件之间进行通信的机制，而无需这些组件之间存在直接的耦合。这种模式通过发布-订阅模式实现，允许一个组件（发布者）发布事件，而其他感兴趣的组件（订阅者）可以订阅这些事件并做出相应的响应。

### 事件聚合器的核心概念

* **发布者（Publisher）**：产生事件的组件。
* **订阅者（Subscriber）**：对特定事件感兴趣的组件。
* **事件（Event）**：发布者发布的信息，订阅者订阅的对象。
* **事件聚合器**：一个中央枢纽，负责管理事件的发布和订阅。

### 事件聚合器的优势

* **松耦合**：组件之间不需要知道彼此的存在，只需要关注事件本身。
* **可扩展性**：新的组件可以很容易地订阅已有的事件，或者发布新的事件。
* **可测试性**：组件可以独立地进行测试，因为它们之间的交互是通过事件进行的。
* **复用性**：事件可以被多个订阅者订阅，从而提高代码的复用性。

### 事件聚合器的应用场景

* **用户界面**：不同的用户界面元素可以通过事件进行通信，例如按钮点击事件、文本框内容变化事件等。
* **模块间通信**：不同的模块可以通过事件进行交互，例如数据加载完成事件、错误发生事件等。
* **异步操作**：事件可以用于在异步操作完成后通知其他组件。

### 事件聚合器的实现

事件聚合器的实现方式有很多种，但它们的基本原理都是相同的。通常，事件聚合器会提供以下接口：

* **订阅事件**：订阅者通过调用事件聚合器的订阅方法来订阅特定的事件。
* **发布事件**：发布者通过调用事件聚合器的发布方法来发布事件。
* **取消订阅**：订阅者可以随时取消对某个事件的订阅。

下面我们基于之前的代码添加事件聚合器。

## 二、添加DTO

DTO（Data Transfer Object，数据传输对象）。

新建一个Dto来传输事件的内容信息。

在`Core`类库中，新增文件夹Dtos，新增类`EventTestDto`，新增两个自动属性，代码如下

```
using Core.Abstracts;

namespace Core.Dtos
{
    public class EventTestDto : IEventTest
    {
        #region Implementation of IEventTest

        /// <summary>Initializes a new instance of the <see cref="T:System.Object" /> class.</summary>
        public EventTestDto(string title, string content)
        {
            Title = title;
            Content = content;
        }

        public string Title { get; set; }
        public string Content { get; set; }

        #endregion
    }
}

```

在这个类中，我们添加了两个`string`类型的自动属性，`Title、Content`,并添加了构造函数。IEventTest是这两个字段的接口，这里使用接口是为了方便后面接入数据库做准备。

代码如下

```
namespace Core.Abstracts
{
    public interface IEventTest
    {
        string Title { get; set; }

        string Content { get; set; }
    }
}

```

## 三、新增EventTest事件

在`Core`类库中新增Events文件夹，并新增类文件EventTest，继承自类`PubSubEvent`代码如下

```
using Core.Dtos;

namespace Core.Events
{
    public class EventTest : PubSubEvent<EventTestDto>
    {
    }
}

```

## 四、添加事件发出者和订阅者

### 修改应用程序布局

为了更直观的查看事件聚合器带来的效果，我们先修改应用程序布局

在`MauiPrism9Demo`中，修改`MainPage`文件，修改后的代码如下

```
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage x:Class="MauiPrism9Demo.MainPage"
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:core="clr-namespace:Core;assembly=Core"
             xmlns:prism="http://prismlibrary.com">

    <Grid RowDefinitions="*,*">
        <ContentView Grid.Row="0" prism:RegionManager.RegionName="{x:Static core:RegionNames.MainRegion}" />
        <ContentView Grid.Row="1" prism:RegionManager.RegionName="{x:Static core:RegionNames.ContentRegion}" />
    </Grid>



</ContentPage>

```

这里是将原来的只有一个区域变成两个等分的区域，其中的`RegionNames`是为了方便，以及导航的准确性，新增的导航区域标注类，这个类也是放在`Core`中，代码如下

```
namespace Core
{
    public static class RegionNames
    {
        public const string MainRegion = nameof(MainRegion);
        public const string ContentRegion = nameof(ContentRegion);
    }
}

```

### 修改UIModule

原来我们使用ViewA作为应用程序的首页，现在我们修改UIModule，同时将ViewA和ViewB都显示出来。

在`UIModule`的`OnInitialized`方法中，新增导航将ViewB导航到ContentRegion，代码如下

```
  regionManager.RegisterViewWithRegion(RegionNames.ContentRegion, nameof(ViewB));
```

完整的UiModule代码如下

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
            regionManager.RegisterViewWithRegion(RegionNames.ContentRegion, nameof(ViewB));
        }

        #endregion
    }
}
```

此时运行程序，不出意外的话，ViewA和ViewB应该是同时出现的应用程序中的。

### 发布事件

现在，我们在ViewA中新增按钮，点击之后发布事件。在ViewAViewModel中新增发布按钮点击后发布事件的Command。

ViewA的代码如下

```
<?xml version="1.0" encoding="utf-8" ?>
<ContentView x:Class="UI.Views.ViewA"
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

        <Button Command="{Binding PublishEventCommand}"
                HorizontalOptions="Center"
                Text="PublishEvent"
                VerticalOptions="Center" />
    </VerticalStackLayout>
</ContentView>

```

ViewAViewModel的代码如下

```
using System.Windows.Input;
using Core;
using Core.Dtos;
using Core.Events;
using UI.Views;

namespace UI.ViewModels
{
    public class ViewAViewModel(IRegionManager regionManager, IEventAggregator eventAggregator) : BindableBase
    {
        public ICommand ToViewBCommand => new DelegateCommand(() =>
        {
            regionManager.RequestNavigate(RegionNames.MainRegion, nameof(ViewB));
        });

        public ICommand PublishEventCommand => new DelegateCommand(() =>
        {
            eventAggregator.GetEvent<EventTest>().Publish(new EventTestDto("这是一个Title",
                $"这是一个Content，发布时间为：{DateTime.Now:yyy-MM-dd HH:mm:ss}"));
        });
    }
}
```

这里ViewA中的PublishEvent按钮点击以后，将会发布一个EventTest事件，带了一个EventTestDto的数据。

事件聚合器`IEventAggregator`通过主构造函数注入。

### 订阅事件

在ViewBViewModel的构造函数中，添加对EventTest事件的监听，代码如下

```
using System.Windows.Input;
using Core;
using Core.Dtos;
using Core.Events;
using UI.Views;

namespace UI.ViewModels
{
    public class ViewBViewModel : BindableBase
    {
        private readonly IRegionManager _regionManager;

        public ViewBViewModel(IRegionManager regionManager, IEventAggregator eventAggregator)
        {
            _regionManager = regionManager;

            eventAggregator.GetEvent<EventTest>().Subscribe(OnEventTest);
        }

        private void OnEventTest(EventTestDto dto)
        {
            TestDto = dto;
        }

        private EventTestDto _testDto;

        public EventTestDto TestDto
        {
            get => _testDto;
            set => SetProperty(ref _testDto, value);
        }


        public ICommand GoBackCommand => new DelegateCommand(() =>
        {
            _regionManager.RequestNavigate(RegionNames.MainRegion, nameof(ViewA));
        });
    }
}

```

这里我们使用了构造函数，并在其中添加了EventTest的事件监听，然后使用EventTestHandle方法来处理事件，这里是将事件Dto赋值给了一个本地属性。我们在ViewB视图中将内容显示出来，代码如下

```
<?xml version="1.0" encoding="utf-8" ?>
<ContentView x:Class="UI.Views.ViewB"
             xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml">

    <Grid>
        <VerticalStackLayout HorizontalOptions="Center" VerticalOptions="Center">
            <Label HorizontalOptions="Center"
                   Text="ViewB"
                   VerticalOptions="Center" />
            <Button Command="{Binding GoBackCommand}"
                    HorizontalOptions="Center"
                    Text="GoBack"
                    VerticalOptions="Center" />

            <StackLayout Orientation="Horizontal">
                <Label Text="Title:" />
                <Label Text="{Binding TestDto.Title}" />
            </StackLayout>
            <StackLayout Orientation="Horizontal">
                <Label Text="Content:" />
                <Label Text="{Binding TestDto.Content}" />
            </StackLayout>

        </VerticalStackLayout>
    </Grid>
</ContentView>


```

此时运行程序，点击按钮，就可以直观的看到事件聚合器的效果了，我们在ViewAViewModel发布的事件被ViewBViewModel监听到，并显示在界面ViewB界面上。

### 解除订阅

为了避免出现不可预知的问题，我们再不需要处理事件的时候，应该把已经订阅的事件取消，代码如下

```
 _aggregator.GetEvent<EventTest>().Unsubscribe(EventTestHandle);
```

自此，完整的事件聚合器闭环就完成了。

---
title: 从零开始搭建Maui框架05-添加自定义Dialog弹窗
date: 2024-11-19 14:20:32
categories:
	- 教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---
## MAUI 自带的弹窗类型及用法

.NET MAUI (多平台应用 UI) 提供了多种内置的弹窗类型，方便开发者在应用程序中与用户进行交互。这些弹窗在不同的平台上会呈现为原生控件，保证了跨平台应用的一致性。

### 常用的三种弹窗类型

* **DisplayAlert:** 用于显示简单的警报框，通常包含一个标题、一个消息和一个“确定”按钮。
* **DisplayActionSheet:** 提供多个选项供用户选择，类似于 iOS 上的 Action Sheet 或 Android 上的 Bottom Sheet。
* **DisplayPromptAsync:** 用于获取用户输入，可以设置输入类型（文本、密码等）。

Maui提供的弹窗只有简单的功能，而且UI无法定制。

Prism框架自带了`IDialogService` ,可以满足自定义弹窗的需求。

## 创建Dialog视图

Prism的Dialog弹窗视图没有特殊要求，使用`ContentView`即可，根据需求自定义视图UI。

## 创建Dialog视图模型

Dialog的视图模型需要继承`BindableBase` 基类、并实现`IDialogAware`接口。`IDialogAware`接口中要实现的方法如下

```
 /// <summary>
 /// Evaluates whether the Dialog is in a state that would allow the Dialog to Close
 /// </summary>
 /// <returns><c>true</c> if the Dialog can close</returns>
 bool CanCloseDialog();

 /// <summary>
 /// Provides a callback to clean up resources or finalize tasks when the Dialog has been closed
 /// </summary>
 void OnDialogClosed();

 /// <summary>
 /// Initializes the state of the Dialog with provided DialogParameters
 /// </summary>
 /// <param name="parameters"></param>
 void OnDialogOpened(IDialogParameters parameters);

 /// <summary>
 /// The <see cref="DialogCloseListener"/> will be set by the <see cref="IDialogService"/> and can be called to
 /// invoke the close of the Dialog.
 /// </summary>
 DialogCloseListener RequestClose { get; }
```

* `CanCloseDialog` : 指示当前Dialog是否可以关闭
* `OnDialogClosed` : 当Dialog关闭时触发
* `OnDialogOpened` : 当Dialog打开时触发，`parameters`是弹窗时外部传进来的参数
* `RequestClose`: `RequestClose` 的本质是一个Action，执行此方法时关闭当前的Dialog

## 注册Dialog

在调用弹窗之前，需要先将视图(`View`)和视图模型(`ViewModel`)注入到容器中，注册代码如下

```
 containerRegistry.RegisterDialog<xxxDialog, xxxDialogViewModel>();
```

## 弹出Dialog

在需要调用弹窗的地方，使用依赖注入的方法注入`IDialogService`服务，然后调用ShowDialog。

```
  dialogService.ShowDialog(
      name: nameof(xxxDialog),
      parameters: new DialogParameters
      {
          { "key", "value" }
      }, callback: result => { Debug.WriteLine(result); });
```

ShowDialog的参数

* name：视图名称
* parameters：传送给Dialog的参数，在Dialog的OnDialogOpened中可以拿到
* callback：关闭Dialog时，Dialog回传的参数，其中包含`Exception` `ButtonResult` `Parameters`
  * Exception： Dialog中发生的异常
  * ButtonResult：Dialog是通过什么方式关闭的
  * Parameters：Dialog回传的参数，以键值对的形式

## 最后

[Prism官网](https://docs.prismlibrary.com/docs/)

[源码](https://github.com/user121238/MauiPrism9Demo)

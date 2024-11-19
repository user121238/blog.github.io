---
title: 从零开始搭建Maui框架07-多主题切换
date: 2024-11-19 14:25:32
categories:
	- MAUI教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---

Maui的多主题切换，官方给了例子

[官方文档](https://learn.microsoft.com/zh-cn/dotnet/maui/user-interface/theming?view=net-maui-8.0)

### 添加主题文件

* 新增`ResourceDictionary` 命名LightTheme
* 新增`ResourceDictionary` 命名DarkTheme

### 设置默认主题

将上面创建的任一主题资源文件合并到`App.xaml`中

### 使用主题资源

再依赖属性上，使用` DynamicResource`拓展标记使用主题资源

```
 <Style x:Key="LargeLabelStyle"
               TargetType="Label">
            <Setter Property="TextColor"
                    Value="{DynamicResource SecondaryTextColor}" />
            <Setter Property="FontSize"
                    Value="30" />
        </Style>
```

### 在运行时加载主题

在运行时选定了主题时，应用应执行以下操作：

1. 从应用中删除当前主题。 这是通过清除应用级别 [ResourceDictionary](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.maui.controls.resourcedictionary) 的 `MergedDictionaries` 属性实现的。
2. 加载所选主题。 这是通过将所选主题的实例添加到应用级别 [ResourceDictionary](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.maui.controls.resourcedictionary) 的 `MergedDictionaries` 属性来实现的。

然后，任何使用 [`DynamicResource`](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.maui.controls.xaml.dynamicresourceextension) 标记扩展设置属性的 [VisualElement](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.maui.controls.visualelement) 对象都将应用新的主题值。 这是因为 [`DynamicResource`](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.maui.controls.xaml.dynamicresourceextension) 标记扩展要保留一个到字典键的链接。 因此，当替换与键关联的值时，会将更改应用于 [VisualElement](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.maui.controls.visualelement) 对象。

```
ICollection<ResourceDictionary> mergedDictionaries = Application.Current.Resources.MergedDictionaries;
if (mergedDictionaries != null)
{
    mergedDictionaries.Clear();
    mergedDictionaries.Add(new DarkTheme());
}
```

> 上一章使用的UraniumUI使用的是静态资源，使用这种方式不能修改UraniumUI控件的主题资源

上一章

[从零开始搭建Maui框架06-使用第三方UI框架](https://zhuanlan.zhihu.com/p/719308343)

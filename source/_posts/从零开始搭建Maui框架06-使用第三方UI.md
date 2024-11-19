---
title: 从零开始搭建Maui框架06-使用第三方UI
date: 2024-11-19 14:22:32
categories:
	- 教程
tags:
	- C#
	- .NET
	- MAUI
	- Prism
---
.NET MAUI (Multi-platform App UI) 虽然提供了丰富的原生控件，但有时为了实现更复杂、更个性化的用户界面，或者加速开发过程，引入第三方 UI 控件库是一个不错的选择。

### 常用的收费 MAUI 第三方 UI 控件库

* **Syncfusion Essential Studio for MAUI:**

  * 提供了丰富的控件，包括数据网格、图表、日历、仪表盘等。
  * 支持多种平台，并提供了示例和文档。
  * 社区版免费
  * [Syncfusion官网](https://www.syncfusion.com/maui-controls)
* **Telerik UI for MAUI:**

  * 提供了类似的功能，并强调性能和易用性。
  * [Telerik官网](https://www.telerik.com/maui-ui)
* DevExpress：

  * 提供了丰富的组件，更偏向重型应用。.
  * MAUI Control 免费
  * [DevExPress官网](https://www.devexpress.com/maui/)

### 开源的MAUI第三方UI控件库

* [UraniumUI](https://github.com/enisn/UraniumUI)

  * github开源
  * 一直在更新
* [MDC-MAUI](https://github.com/mdc-maui/mdc-maui)

  * Material风格的控件
  * github开源
  * 已经停止维护
* [Xceed-Toolkit-for-.NET-MAUI](https://github.com/xceedsoftware/Xceed-Toolkit-for-.NET-MAUI)

  * 老牌控件库提供商
  * MAUI模块开源免费
  * github开源

### 引入UraniumUI库

虽然第三方库很多，但是这里我们使用更新比较频繁的UraniumUI。

[官网文档](https://enisn-projects.io/docs/en/uranium/latest/Getting-Started)

* 引入Nuget包，现在项目只需要引入`UraniumUI.Material、UraniumUI.Icons.MaterialIcons` 即可
* 在`MauiProgram`的`CreateMauiApp`方法中添加使用UI库的代码 `builder.UseUraniumUI().UseUraniumUIMaterial();`
* 在`MauiProgram`的`CreateMauiApp`方法中添加`MaterialIconFonts`的使用在Fonts的配置中。`fonts.AddMaterialIconFonts();`
* 在`App.xaml`中引入命名空间 ` xmlns:material="http://schemas.enisn-projects.io/dotnet/maui/uraniumui/material"`
* 给自带的Colors和Styles资源命名，并重写资源库

  ```
   <ResourceDictionary x:Name="AppColors" Source="Resources/Styles/Colors.xaml" />
   <ResourceDictionary x:Name="AppStyles" Source="Resources/Styles/Styles.xaml" />
  <material:StyleResource BasedOn="{x:Reference AppStyles}" ColorsOverride="{x:Reference AppColors}" />

  ```

### 使用UI库 模仿QQ登录界面

QQ9的手机版登录界面如下图所示

* 新建一个名为LoginPage的`ContentView`
* 新建一个名为LoginPageViewModel的类，继承自`BindableBase`
* 在LoginPage的隐藏代码`LoginPage.xaml.cs`中，添加Ioc注入

  ```
  using Core.Attributes;
  using UI.ViewModels;

  namespace UI.Views;

  [IocForRegionNavigation(typeof(LoginPageViewModel))]
  public partial class LoginPage : ContentView
  {
      public LoginPage()
      {
          InitializeComponent();
      }
  }
  ```
* 在LoginPage中引入命名空间` xmlns:material="clr-namespace:UraniumUI.Material.Controls;assembly=UraniumUI.Material"`

  > 注意，这里千万不要引错了，这里的命名空间是带Controls的。`http://schemas.enisn-projects.io/dotnet/maui/uraniumui/material`是错误的命名空间，VS编译通不过。
  >
* 修改LoginPage的代码

  ```
  <?xml version="1.0" encoding="utf-8" ?>
  <ContentView x:Class="UI.Views.LoginPage"
               xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
               xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
               xmlns:m="clr-namespace:UraniumUI.Icons.MaterialIcons;assembly=UraniumUI.Icons.MaterialIcons"
               xmlns:material="clr-namespace:UraniumUI.Material.Controls;assembly=UraniumUI.Material">

      <Grid RowDefinitions="1*,3*,1*">
          <Grid.Background>
              <LinearGradientBrush StartPoint="0,0" EndPoint="0.5,0.5">
                  <GradientStop Offset="0.1" Color="#D8D9FA" />
                  <GradientStop Offset="1.0" Color="White" />
              </LinearGradientBrush>
          </Grid.Background>

          <!--  MAUI原生不支持渐变色文本，要么使用SkiaSharp手动绘制，要么使用矢量图片代替  -->
          <!--<Label FontSize="40"
                 HorizontalOptions="Center"
                 Text="QQ9"
                 TextColor="AntiqueWhite"
                 VerticalOptions="Center" />-->
          <Image Grid.Row="0"
                 HeightRequest="36"
                 Source="QQ.png" />


          <VerticalStackLayout Grid.Row="1"
                               Margin="32,0"
                               Spacing="30"
                               VerticalOptions="Center">
              <material:TextField Title="输入QQ号"
                                  AllowClear="True"
                                  HorizontalTextAlignment="Center" />

              <material:TextField Title="输入QQ密码"
                                  HorizontalTextAlignment="Center"
                                  IsPassword="True" />


              <material:ButtonView HeightRequest="52">
                  <Label HorizontalOptions="Center"
                         Text="登录"
                         TextColor="White"
                         VerticalOptions="Center" />
              </material:ButtonView>

              <material:CheckBox HorizontalOptions="Center" Text="我已阅读并同意服务协议和QQ隐私保护指引" />

          </VerticalStackLayout>

          <VerticalStackLayout Grid.Row="2"
                               HorizontalOptions="Center"
                               VerticalOptions="Center">

              <material:ButtonView BackgroundColor="White"
                                   HeightRequest="40"
                                   Stroke="Gray"
                                   StrokeThickness="1"
                                   WidthRequest="40">
                  <Image>
                      <Image.Source>
                          <FontImageSource FontFamily="MaterialRegular"
                                           Glyph="{x:Static m:MaterialRegular.Phone}"
                                           Color="Gray" />
                      </Image.Source>
                  </Image>
              </material:ButtonView>
              <Label Margin="0,10,0,0" Text="手机号登录" />
          </VerticalStackLayout>





      </Grid>
  </ContentView>


  ```

  自此，一个低仿的QQ登录就完成了，其中使用了UraniumUI的文本输入框、按钮以及图标文字。

### 总结

引入第三方的UI控件相对来说是比较简单的，现在的第三方控件都有相对完整的文档，对着官网文档即可实现大多数功能，有问题可以直接在github上题issues。

UraniumUI的使用就是：

1. 导入正确的库
2. 在程序启动之前配置
3. 导入正确的资源和命名空间
4. 正常使用

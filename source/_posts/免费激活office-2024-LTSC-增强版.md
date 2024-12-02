---
title: 免费激活office 2024 LTSC 增强版
tags:
  - office
  - 激活
categories:
  - 工具
date: 2024-12-02 11:11:13
---
# 免费激活Office 2024 LTSC 专业增强版

1、 下载office软件安装工具

下载链接

```
https://www.microsoft.com/en-us/download/details.aspx?id=49117
```

![alt text](images/免费激活office-2024-LTSC-增强版/image.png)

2、 下载配置文件

下载链接

```
https://config.office.com/deploymentsettings
```

进入页面后，先配置左侧部署设置，其中

* 产品/Office套件 选择`Office LTSC Professional Plus 2024 - Volume License`
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-1.png)
* 应用中的选项根据自己的需要选，选的越多，下载时间越长（废话）
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-3.png)
* 更新和升级选项 中 选择 `Office内容分发网络(CDN)`
* 如果以前安装过其他版本的Office,建议将`卸载所有 Office MSI 版本，包括 Visio 和 Project`开启
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-2.png)
* 授权和激活/产品密钥 选择 `KMS`
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-4.png)
* 其他选项根据自己的需要选择即可
* 所有的选择的完成以后，点击完成
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-5.png)
* 导出配置文件,两个导出按钮都可以
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-6.png)
* 默认格式选择 `保留当前设置` 或者 `Office Open XAML 格式` 都可以
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-7.png)
* 导出配置文件，配置文件名最好不要使用中文
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-8.png)

3、 安装
新建一个文件夹，最好路径不要包含中文，将下载好的`officedeploymenttool_18129-20030.exe` 和 `配置文件` 放在新建的文件夹中。
![alt text](images/免费激活office-2024-LTSC-增强版/image-9.png)

* 双击 `officedeploymenttool_18129-20030.exe` ，勾选左侧的选择框，然后点击右侧的Continue按钮
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-10.png)
  点击按钮之后，会让你选择文件夹，就选择刚刚新建的文件夹就行，之后会在文件夹中生成安装文件
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-11.png)
* 用管理员权限运行 `cmd`
  ![alt text](images/免费激活office-2024-LTSC-增强版/image-12.png)

使用以下命令

```
cd [path]
```

[path]为刚刚新建的文件夹的路径，比如在 `C:\Users\Eric\Desktop\office 2024` ,则命令为

```
cd C:\Users\Eric\Desktop\office 2024
```

![alt text](images/免费激活office-2024-LTSC-增强版/image-13.png)

继续使用以下命令安装office

```
setup /configure config.xml
```

就会出现安装界面

![alt text](images/免费激活office-2024-LTSC-增强版/image-14.png)

等待安装完成即可。

不出意外的话，安装完成以后，你的Office就已经处于激活状态了。
如果出了意外，没有激活的话，继续在命令控制台执行以下命令

```
cd C:\Program Files\Microsoft Office\Office16
cscript ospp.vbs /sethst:kms.03k.org 
cscript ospp.vbs /act
```

> 需要注意的事，如果你安装的是32位的话，第一行的命令需要换成 `cd C:\Program Files (x86)\Microsoft Office\Office16 `

执行到这里，Office就已经激活了，可以正常使用！！
![alt text](images/免费激活office-2024-LTSC-增强版/image-15.png)

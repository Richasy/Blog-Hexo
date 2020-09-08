---
title: 在IIS上部署 .Net Core服务
lang: zh
date: 2019/3/25 20:45:12
type: post
categories:
- 网页
- .NET Core
tags:
- iis
- .net core
- web
---

:::tip
如果你不知道怎么在自家电脑上开启IIS服务器，请参照[启用IIS服务器](./open_iis)
:::

> .Net Core是微软出的跨平台框架，彻底让C#摆脱平台束缚，得以大展拳脚。本文便是记录如何在IIS服务器中部署 .Net Core应用，以便于开发测试。

<!--more-->

## 下载与安装

[.Net Core下载地址](https://dotnet.microsoft.com/download)

如果你有兴趣做 .Net Core的开发，并且还具备一定C#的知识，那么可以看微软提供的[文档](https://docs.microsoft.com/en-us/dotnet/core/)

![blog_web_others_netcore_01.png](https://storage.live.com/items/51816931BAB0F7A8!12428?authkey=AO7QXpgYo7-5DUU)

进入下载页面后，先别急着下载。如果你是准备部署某个 .Net Core的软件包，那么务必搞清楚它是基于哪个 .Net Core的版本构建的，如果下载了错误的包，那软件是运行不起来的。

截至我写此文时，.Net Core的最新正式版本为2.2系列，如果你手头上的安装包基于更老的版本，需要进入[历史下载](https://dotnet.microsoft.com/download/archives)页面去找寻对应的安装包。

.Net Core提供了两种包，一种是SDK，一种是Runtime，前者用于开发，而后者用于部署。

简单来说，如果你想做 .Net Core开发，那么你需要把两个都下载下来。如果你只为了部署某个 .Net Core应用，那么下载Runtime并安装即可。

下载安装的具体步骤不表，由于我们这次主要是做IIS服务器的部署，那么下载Windows平台对应的运行包即可。

:::warning
.Net Core在安装环境包时也会出现一些奇怪的问题，这些问题往往出现在低版本的Windows上，比如Win7或者Windows Server 2008。这些系统在安装时可能会报错，其原因往往是补丁没有打上，或者系统版本不满足最低要求。

具体的支持版本和遇到安装问题的解决方法点[这里](https://docs.microsoft.com/en-us/dotnet/core/windows-prerequisites?tabs=netcore2x)
:::

## 如何在IIS上部署

我默认你已经启动了IIS服务器了。

.Net Core服务的部署和寻常 asp .net服务还不太一样，我们需要先构建一个应用池。

### 构建应用池

打开IIS管理器，在`应用程序池`上右键，添加一个应用程序池，并按图中所示填写。

![blog_web_others_netcore_02.png](https://storage.live.com/items/51816931BAB0F7A8!12429?authkey=AO7QXpgYo7-5DUU)

填写完成后点击确定。

### 添加网站

不管你的 .Net Core程序是什么，Web Api也好，网页也罢。总之，要部署就先新建一个网站。

在`网站`上右键新建一个网站，根据自己的需求填写对应项，记得将应用程序池选为自己刚建的程序池。

![blog_web_others_netcore_03.png](https://storage.live.com/items/51816931BAB0F7A8!12430?authkey=AO7QXpgYo7-5DUU)

点击确定，如此网站就建成了。

现在，只要你顺利安装了 .Net Core的Runtime包，你就可以通过 `http://localhost:your_port` 来访问你的网站了。

## 练个手

假使你还没有一个可以部署的 .Net Core软件包，那么可以试试我给你的这个：

[下载地址](https://github.com/Richasy/HTML-Conversion-Api/releases)

你可以下载最新版本的 `Source Code`，虽然名为Source Code，但它其实是个Web Api的部署包，是用来进行HTML转PDF的。

将其下载下来并解压，按照之前的过程针对这个软件建一个网站，然后你可以通过以下路径访问：

==http://your_server/api/values==

如果返回

```json
["value1","value2"]
```

那就说明部署成功啦~

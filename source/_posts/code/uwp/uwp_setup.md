---
title: UWP小书 - Visual Studio的安装与配置
lang: zh
date: 2019/5/2 18:45:12
type: post
categories:
- UWP
tags:
- uwp
---

## UWP与Windows

对于UWP应用，看到这篇博文的人想必都不会陌生，至少用过那么一两个，比如网易云音乐UWP，系统自带的日历、邮件、OneNote等等。它们都有一个特点，那就是只能在Windows10上运行，Windows10以下的系统版本则无缘UWP应用。

可要说我是Windows10，就一定可以运行UWP应用了吗？也不尽然。

Win10作为最后一代Windows，正在以每年两次大更新的频率进行版本迭代，这些大更新中往往会加入新的功能，新的系统API，新的控件……如果你的UWP应用引用了这些新内容，那么低版本的Win10就不在你的应用支持范围内了，除非你做了降级处理。

所以作为UWP应用的开发者，你必须对Win10的版本有着足够的了解，知道从最早的`10240`版本，到目前最新的`17763`（截至2019/3/19），它们有哪些追加的控件，有哪些新的API等。

不要为难，说这些不是真的要你背，只为了让你了解，UWP应用和Windows版本联系密切，作为开发者不可不察。

作为UWP开发者，咱们的基本原则是用新不用旧，但要求稳。

你可以保持自己的Windows系统最新，但不要强上预览版，最好有一个低版本的虚拟机用以开发测试。在建立新项目时，以低于最新版本一个版本号为宜，在保证功能的同时，尽量扩大潜在受众。

<!--more-->

## 安装Visual Studio

Visual Studio下载地址：[这里](https://visualstudio.microsoft.com/)

Visual Studio作为UWP应用开发的唯一指定IDE，目前已经出到了2019版本，但由于其还处于预览阶段，我们这里就以2017版本举例吧。

学习阶段，我们下载 `Community`版本。

![uwp_setup_01.png](https://storage.live.com/items/51816931BAB0F7A8!12434?authkey=AO7QXpgYo7-5DUU)

下载之后启动安装包，稍作等待，你会看到这样的界面：

![uwp_setup_02.png](https://storage.live.com/items/51816931BAB0F7A8!12435?authkey=AO7QXpgYo7-5DUU)

到了这一步，务必慎重。检查你的网络连接，确保你有足够的时间下载。因为接下来你可能要下载10-20G左右的内容。

为了以后可能接触到的开发内容，我建议你在选择 **通用Windows平台开发** 的同时，将 **.Net桌面开发** 、**ASP .Net Web开发**、**.Net Core** 开发一并选上，如果你无意做这些开发，那么仅选中**通用Windows平台开发**也无妨。

此时不要急着下一步，在右侧点开通用Windows平台开发，勾选你可能用上的SDK

![uwp_setup_03.png](https://storage.live.com/items/51816931BAB0F7A8!12437?authkey=AO7QXpgYo7-5DUU)

如果你的C盘空间不足，记得调整下载位置。

在此之后，就可以开始下载安装了。

更为详细的安装步骤请参见[延伸阅读](#延伸阅读)

---

## 小结

本章主要介绍UWP应用和Windows平台的关系，以及开发IDE-`Visual Studio`的下载及安装。

## 延伸阅读

- [什么是通用 Windows 平台 (UWP) 应用？](https://docs.microsoft.com/zh-cn/windows/uwp/get-started/universal-application-platform-guide)
- [安装 Visual Studio 2017](https://docs.microsoft.com/zh-cn/visualstudio/install/install-visual-studio?view=vs-2017)

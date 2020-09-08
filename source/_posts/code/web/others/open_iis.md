---
title: 启用IIS服务器
lang: zh
date: 2019/3/25 20:45:12
type: post
categories:
- 网页
tags:
- iis
- web
---

> 本文记录如何在Windows系统中启用IIS服务器，通过IIS服务器，可以将本机变作一个服务器，方便一些操作。

<!--more-->

1. ## 找到入口

    进入`控制面板`，找到`程序和功能`，在左侧面板中选择`启用或关闭Windows功能`

    ![blog_web_others_iis_01.png](https://storage.live.com/items/51816931BAB0F7A8!12425?authkey=AO7QXpgYo7-5DUU)

2. ## 打开功能

    之后，按照图中所示，找到 `Internet Information Services` ，并勾选相应模块

    ![blog_web_others_iis_02.png](https://storage.live.com/items/51816931BAB0F7A8!12426?authkey=AO7QXpgYo7-5DUU)

    :::warning
    你至少需要选中 `应用程序开发功能` 这个模块内的部分功能。当然，全选也无所谓。
    :::

3. ## 安装及启动

    选好之后，点击确定，等待Windows安装完成即可。

    现在，IIS服务器已经启动完成，你可以直接在系统中搜索 `IIS`就可以打开了。

    ![blog_web_others_iis_03.png](https://storage.live.com/items/51816931BAB0F7A8!12427?authkey=AO7QXpgYo7-5DUU)

    现在，在浏览器中访问 `http://localhost` ，就可以看到一个默认的网页。出现这个网页就意味着你的服务器已经搭建好啦~只不过目前只有你能访问而已。

---

*本文不过是个开胃小菜，开启IIS服务器不难，开启之后怎么把玩才是关键。*


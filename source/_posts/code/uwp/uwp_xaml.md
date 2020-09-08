---
title: UWP小书 - 谈谈XAML
lang: zh
date: 2019/5/3 20:45:12
type: post
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- XAML简介
- XAML的相关元素，如命名空间
- XAML与CS文件的关连原理
:::

经过前面创建Hello World应用，相信你现在已经对UWP应用开发有了一定的了解。只要揭开那层神秘的面纱，UWP应用开发也并非可望不可及。

但我们的教学终究不会以那种我说什么你做什么的方式进行下去，现在，携着你之前写APP的经验，我们来了解一些理论知识，讲一讲XAML。

<!--more-->

## 什么是XAML

XAML，全称 `Extensible Application Markup Language` ~~奇怪，这不应该叫EAML吗？~~。XAML在2006年之后被广泛应用于微软的各种开发平台上，WPF、UWP、Sliverlight等等。

XAML本身是特化的XML，所以它也遵循XML的语法规则。比如，每个XAML标签包含一个名称和多个属性，而且我之前说过，XAML的每个标签都对应着 .NET中的一个类，标签的属性也和类的属性一一对应。这意味着，只要你想，丢掉XAML，纯用C#手撸界面也不是不可能的。

那我们为什么要用XAML？

为了分离。

UI代码一旦与逻辑代码混杂在一起，软件将变得非常臃肿且难以维护。这一点，在我们之后学习动画时你会有体会。

现在让我们来看一段XAML代码：

```xml
<Button Name="MyButton" Click="MyButton_Click" Content="点我" />
```

这段代码不陌生吧，你不久前才写过。

上面的这段代码中的按钮`Button`，实际上是 Win10中的 `Windows.UI.Xaml.Controls.Button` 类，我们讲过，XAML中的属性和所属类的属性一一对应，所以代码中的 `Name`，就是Button的一个属性。

在这段XAML代码中，还有一个事件处理程序`MyButton_Click`。XAML是支持直接在标签内创建处理程序的，而它对应的具体处理逻辑，在`.xaml.cs`的后台代码文件中，它会生成一个`MyButton_Click`方法。

这个`.xaml.cs`，就是我在[建立第一个应用](./uwp_newapp.html#为什么软件打开之后会显示MainPage)提及的`Code-Behind`。关于当前XAML文件的所有事件代码都在这里。.xaml和.xaml.cs，两者一一对应，但又不总是如此，比如我们以后会介绍到的资源字典。

XAML相比起XML或HTML，要严格许多，这种严格有时候甚至有些麻烦，但却足够井然有序和可靠，比如标签名首字母必须大写啦，所有元素都必须是封闭的啦，诸如此类还有很多，具体内容可以在[延伸阅读](#延伸阅读)中查看。

## XAML元素

### 命名空间

在最开始新建Hello World应用时，我曾告诉你，忽略掉Page标签内的一长串内容，如果那时你心存疑惑，那么现在就让我来告诉你，这里面是什么吧。

我们取Page标签内的某一段来解释一下：

```xml
<Page
    ...
    x:Class="Hello_World.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Hello_World"
    ...
>
</Page>
```
学习过C#你会知道，C#中有一个关键字`namespace`，它表示命名空间。当存在两个同名的类时，往往会通过命名空间进行区分，而与 .NET的类关系密切的XAML，同样也有自己的命名空间。

就比如上面代码中有一段`xmlns:local="using:Hello_World"`，这就表示当前引用了应用程序里的`Hello_World`命名空间，并给它取名叫`local`，这样，在XAML中，你就可以通过 `<local:XXX></local:XXX>`来使用`Hello_World`命名空间下的控件或者类。

知道了这一点，那我问你一个问题，为什么我们写`Button`之类的控件不需要加上命名空间前缀呢？难道微软不担心我会创建一个同名控件吗？

假设我在应用程序内创建了一个控件也叫`Button`，那当我在XAML敲下Button控件时，应用程序到底会用哪个控件呢？

想要知道这一点，我们要看看这两个特殊的命名空间：

```
xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
```

你创建的每一个XAML界面文件都会包括这两个命名空间。

`xmlns=http://schemas.microsoft.com/winfx/2006/xaml/presentation`就是Win10界面的核心命名空间了，这里面包括了绝大多数用来构成界面的控件类，任何没有前缀的控件都会被归类于这个命名空间下。

`xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"`重要性不亚于上面的命名空间，你能看到，这个命名空间被定义为`x`，你在以后会接触到的诸如`x:Name`，`x:Bind`等都归类于这个命名空间下，这个命名空间包含了XAML的诸多实用特性。

看到这里你就知道，Page自带的这些家伙是构建应用的基石，它本质上和C#文件头顶的那一大串`using`差不多，都是引入一些依赖。

但不得不说，这东西的语法看起来真的很怪异，微软的东西几乎都采用帕斯卡(pscal)命名(即单词首字母大写，无空格)，而XAML的命名空间不大写不说，还有一堆不明所以的链接，甚至还不能用浏览器打开。这完全不符合`和.Net类一一对应`的原则嘛。

XAML的命名空间之所以会变成这样，一方面是沿袭XML的命名空间规范（命名不规范，作者两行泪），另一方面是为了降低XAML的复杂度，试想，如果每个控件的命名空间都不相同，那你每写一个控件都要加上前缀，并在头部加入命名空间，想想就觉得毛骨悚然（看看隔壁安卓的XML界面开发，连写个属性都要加前缀）。

### 对象

XAML中的对象指的是XAML中的一个完整的节点。

在XAML中，只允许有一个根节点，在UWP的界面XAML中，通常是`Page`。

在XAML中，除了根节点，其它都是子节点，理论上，子节点无上限，而写节点的语法跟XML无二致。由`<`开始，到`>`收尾。前面说过，XAML要求所有节点都是封闭的，但封闭节点，在XAML当中却有两种写法：

- 常规：`<Button>点我</Button>`
- 自封闭：`<Button Content="点我" />`

### 附加属性

附加属性是一种比较特殊的属性，它是由父节点附加到子节点上的属性，子节点本身没这个属性。

举个例子，比如`Grid`控件。我们知道，`Grid`是表格的意思，既然是表格，那就有行有列，如果我们要写一个两行两列的表格，我们可以这么写：

```xml
<Grid>
   <Grid.RowDefinitions>
        <RowDefinition />
        <RowDefinition />
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition />
        <ColumnDefinition />
    </Grid.ColumnDefinitions>
</Grid>
```

定义了行列之后，我们可以把控件放进去，这个时候，我们就可以给控件定义它是第几行第几列。如果我们准备将按钮放在第二行第二列(右下)，那么我们可以这样写：

```xml
<Grid>
   <Grid.RowDefinitions>
        <RowDefinition />
        <RowDefinition />
    </Grid.RowDefinitions>
    <Grid.ColumnDefinitions>
        <ColumnDefinition />
        <ColumnDefinition />
    </Grid.ColumnDefinitions>
    <Button Grid.Row="1" Grid.Column="1" Content="点我" />
</Grid>
```

这里的`Grid.Row`和`Grid.Column`就是Grid附加到Button上的属性了，而Button本身是没有`Row`和`Column`属性的。这就是容器控件会对子元素造成的影响，在以后讲排版布局时，我们会更深入地讲解。

### 标记扩展

标记扩展是XAML的一个非常骚气的功能。简而言之，你可以在别处定义一大堆XAML逻辑，或者在代码中写一堆逻辑，然后通过标记扩展直接附加到XAML上，从而让布局更为灵活。

我们举个简单的例子，比如我有十来个TextBlock控件，我想给它们设定统一的样式：黑体，加粗，30号字。我该怎么办，一个控件一个控件去写属性吗？那太累了。

我可以先把样式定义好（具体定义在哪，后续会讲）：

```xml
<Style TargetType="TextBlock" x:Key="MyBlock">
    <Setter Property="FontSize" Value="30"/>
    <Setter Property="FontFamily" Value="黑体"/>
    <Setter Property="FontWeight" Value="Bold"/>
</Style>
```

这样我就可以直接在控件中调用

```xml
<TextBlock Style="{StaticResource MyBlock}" Text="Hello World" />
```

这样就可以直接为控件附加样式了。

当然，这只是一点粗浅的应用，只是简化了XAML写作，除了StaticResource，还有其它的扩展标记

- `Binding`：绑定，在运行时将资源或数据绑定到XAML对象，`x:Bind`是在编译时绑定
- `StaicResource`：静态资源，引用事先定义好的资源
- `ThemeResource`：主题资源，可以根据Win10主题变化而变化
- `TemplateBinding`：模板绑定，至于什么是模板，你可以理解为一个独立的XAML片段
- `RelativeSource`：绑定关联源，对特定的数据进行绑定

在XAML中，会使用大括号`{}`来表示标记扩展，就如我在上面写的引用样式一样，这是特定语法，请牢记。

在后续的课程中，我会为大家介绍相关的绑定。

## XAML编译

XAML和其附属的`.xaml.cs`一同构成了界面文件，而软件的运行，就是将这两者混合拍在一起，跟其它的界面文件一起咽下去，经过某种奇怪的过程（编译），最后拉出来放在盘里递到用户面前，用户就开始**愉悦**地享受了。

不开玩笑了，XAML的编译还是有点意思的，我们以MainPage.xaml举例。

如果你的C#水平比较高，可能注意到，在MainPage.xaml.cs里，MainPage类的定义用了一个关键字`partial`，这个关键字往往表示该类在程序集的其它地方也有定义，简而言之，就是MainPage类被分布到了其它文件中。

但是你翻遍了项目里的文件，好像找不到哪个地方还定义了MainPage，难道这是微软的一个恶作剧吗？

非也。

我们在文件管理器中打开项目文件夹，你会发现除了你能在VS中看到的内容外，还有两个文件夹，分别是`bin`和`obj`。打开obj文件夹，在对应平台的`Debug`目录下你就能找到`MainPage.g.cs`(在此之前你必须先成功生成一次项目)。

这个`.g.cs`文件就是我们要找的东西了，打开它看看：

```csharp
partial class MainPage : 
        global::Windows.UI.Xaml.Controls.Page, 
        global::Windows.UI.Xaml.Markup.IComponentConnector,
        global::Windows.UI.Xaml.Markup.IComponentConnector2
    {
        /// <summary>
        /// Connect()
        /// </summary>
        [global::System.CodeDom.Compiler.GeneratedCodeAttribute("Microsoft.Windows.UI.Xaml.Build.Tasks"," 10.0.17.0")]
        [global::System.Diagnostics.DebuggerNonUserCodeAttribute()]
        public void Connect(int connectionId, object target)
        {
            switch(connectionId)
            {
            case 2: // MainPage.xaml line 12
                {
                    this.MyTextBlock = (global::Windows.UI.Xaml.Controls.TextBlock)(target);
                }
                break;
            default:
                break;
            }
            this._contentLoaded = true;
        }

        /// <summary>
        /// GetBindingConnector(int connectionId, object target)
        /// </summary>
        [global::System.CodeDom.Compiler.GeneratedCodeAttribute("Microsoft.Windows.UI.Xaml.Build.Tasks"," 10.0.17.0")]
        [global::System.Diagnostics.DebuggerNonUserCodeAttribute()]
        public global::Windows.UI.Xaml.Markup.IComponentConnector GetBindingConnector(int connectionId, object target)
        {
            global::Windows.UI.Xaml.Markup.IComponentConnector returnValue = null;
            return returnValue;
        }
    }
```

到这里，我们就找到了MainPage的另一处定义。

你可能看这堆代码有点懵，不瞒你说，我也是。

这个文件是干嘛的？这里的文件其实就是给应用和你的MainPage页搭了一个桥梁，你在Xaml中命了名的控件会出现在这里，这就是做了个路牌，回想一下，为什么你在XAML文件中给控件命名后就能在C#中获取到该控件？原因就在这儿了，你需要的控件在这里被定义好，所以你才可以引用。

说到这里，我们必须要提及另一个相似的文件：

`MainPage.g.i.cs`

在同一目录下，除了`.g.cs`文件外，还有`.g.i.cs`，你打开看看，发现大同小异，而且这里也是MainPage类的另一处定义，顺带还发现了在`Mainpage.xaml.cs`中自带的InitializeComponent方法。在这个方法中，我们能看到`MainPage.xaml`被加载进来。

## 小结

本章主要讲述XAML相关的知识。篇幅较长，但即便如此，出于教学需要，我也省去了部分内容，比如动态加载XAML，XAML的树结构等。

这并不意味着我不会涉及这些内容，相反，因为这些内容属于比较高级的应用，我会在后面的课程中再详加阐述。

本篇内容比较理论，也许对你而言略显晦涩，莫慌，看不懂没关系，看完之后留个印象就可以。实际上，即便你不看本文，UWP应用还是照样写，但会用和灵活使用是两个概念。

如果你在写了一段时间的UWP应用之后，再回过头看看这些基础的文章，反而会有茅塞顿开之感，越看越带劲，但在初学时却往往觉得艰深无趣，直接跳过，这也没甚打紧，都是这么过来的。现在不喜欢看，留着以后再看吧。

## 延伸阅读

- [XAML概述](https://docs.microsoft.com/zh-cn/windows/uwp/xaml-platform/xaml-overview)

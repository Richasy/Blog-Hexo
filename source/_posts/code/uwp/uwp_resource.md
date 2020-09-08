---
type: post
title: UWP小书 - 样式的作用域以及资源
date: 2019/6/1 10:19
lang: zh
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- 样式的作用域
- 资源是什么
- 主题的切换
:::

视频地址：[BiliBili](https://www.bilibili.com/video/av54257809/)

## 前言

上节课，我们讲述了基础的样式修改，以及公共样式的建立。

在上节课里，我要求大家将建立的Style放在`<Page.Resources>`标签中，为什么要放在这个标签里，我并未详加阐述。本节课，我们就要从这里入手，带你了解样式的作用域。

<!--more-->

## 样式的作用域

样式，说穿了就是部分属性的集合。这个集合我们称之为样式。

写在控件内的，比如在`<Button>`标签内写`FontFamily`属性，这种即为行内样式。它只更改所属控件的样式。我们说它的作用域仅限于所属控件本身。

而如我们上节课做的，将样式抽离出来，放在`<Page.Resources>`中，所有引用该样式的控件都能应用样式。我们说它的作用域限于当前页面内的所有引用控件。

引用控件好说，为什么作用域仅限于当前页面呢？

注意我们放样式的地方：

`Page.Resources`

页面的资源。

这里你就应该明白了。将样式作为资源放到当前页面的资源列表中，可不就是只有当前页面才能引用吗？

当你新建了B页面，再想引用A页面定义的资源，那就引用不到了。

这就出现问题了。

如果我有一个按钮的样式，这个样式我打算应用到所有按钮上，是跨页面的。难道我每新建一个页面就要再写一次按钮样式吗？

不必。

那怎么办？

**扩大作用域**。

应用是一个框，框里面有许多的页面，按照功能进行切换。那么页面的上一个层级是什么？

应用。

换句话说，我们只要将样式定义在应用这个层级上，那么凡是该应用下的页面都可以获取到，这一点是毫无疑问的。

问题是，怎么定义在应用层级上呢？

---

新建应用的时候，VS提供了两个xaml文件，一个叫做MainPage.xaml，另一个叫App.xaml。MainPage是我们的主页，而App则代表了我们的应用。

让我们打开App.xaml。

现在的App.xaml中除了命名空间别无一物。不要紧，就像我们在Page中做的那样，先新建一个Resources标签，然后将我们Page中的Style移过来即可：

```xml
<Application.Resources>
    <Style x:Key="BasicButton" TargetType="Button">
        <Setter Property="FontSize" Value="15"/>
        <!-- 输入其它样式属性 -->
    </Style>
</Application.Resources>
```

现在试一试，不管在哪个页面，所有的Button控件现在都可以引用这个样式了。

这就是我们所谈的作用域。

我们可以发现，样式的作用域很简单，层级也很少。

```
控件 -- 页面 -- 应用
```

就这么简单。

## 谈谈资源

不知你是否有留意，我们存放样式的标签为何叫Resources。

Resources是什么？是资源。

那么什么是资源呢？

大而化之，资源表示应用内的一切可引用的数据，比如图片、数据模板、样式文件、字体、颜色等等。

这些资源不是都放在Resources标签内的。比如Assets文件夹，这个文件夹存在的目的是为了放静态文件资源，比如图片，新建应用时，应用的Logo就放在这个文件夹里面。再比如我们常常看到一个应用，支持多语言，这些文本，也是资源，并不放在Resources标签中，这个我们后面也会讲到。

那么放在Resources标签里的有什么呢？

常见的，比如我们的样式，还有DataTemplate，我们称其为数据模板，这个后面会讲到。还有ControlTemplate，控件的模板，这个我们下节课会说明。

总的来说，资源是多种多样的。而且资源有一个特点，那就是事先就会被定义。

也就是说，在你引用这个资源之前，这个资源必须存在。

所以我们会用`StaticResource`作为关键词去引用我们定义好的资源。

但如果你之前有过预习，会发现除了`StaticResource`之外，还有一种资源的引用方式，叫做`ThemeResource`，这又是什么呢？

## 主题资源

谈到`ThemeResource`，就不得不提一下UWP的主题系统了。

从Windows 10的主题设置就可以看出来，Win10主要有两种主题，`Light`和`Dark`。除此之外还有一种高对比度主题，这里我们略过不表。

我们可以通俗地说一个是白昼主题，一个是黑夜主题。这里的主题不光会影响到系统，也会影响UWP应用。

不客气地说，作为一个UWP应用，适配这两种主题不是一个可选项，而是必选项。

为什么这么说？我们来看一个例子：

我显式设置MainPage的`Background`为白色，新建了一个TextBlock，只设置内容，不设置颜色，则其颜色为默认。

![1c1eeef3-9359-4c4b-bcdf-44131a5aa75f.png](https://storage.live.com/items/51816931BAB0F7A8!13348?authkey=AO7QXpgYo7-5DUU)

在Light主题下，没问题，工作得很正常。

![f13e7d64-2736-4be1-8ec1-343b690db7a8.png](https://storage.live.com/items/51816931BAB0F7A8!13349?authkey=AO7QXpgYo7-5DUU)

但是在Dark主题下，凉凉了，什么都看不见了。

是TextBlock消失了吗？

不是。

是TextBlock的前景色变成白色了，也就相当于隐形了。

为什么TextBlock的颜色会改变呢？

因为默认情况下，TextBlock的前景色是使用`ThemeResource`作为关键词的。

这个时候，ThemeResource的作用也就呼之欲出了。**那就是根据主题的不同，切换不同的资源**。

## 如何建立主题资源

前面我们说过，做一个UWP应用，就必须要对两种主题进行适配。那么我们如何建立两种主题所需要的资源呢？

这里我们就来简单地做一个按钮。这个按钮有一个特点，在Light主题中，它是红色的，在Dark主题中，是蓝色的。

如何做？

打开App.xaml，将之前写的Style代码删除，替换为下面的代码：

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.ThemeDictionaries>
            <ResourceDictionary x:Key="Light">
                <SolidColorBrush x:Key="ButtonBackground" Color="Red"/>
            </ResourceDictionary>
            <ResourceDictionary x:Key="Dark">
                <SolidColorBrush x:Key="ButtonBackground" Color="Blue"/>
            </ResourceDictionary>
        </ResourceDictionary.ThemeDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

我们来解释一下这段代码。

在Resource标签中，我们首先建立一个资源字典 *(Resource Dictionary)* 。Resource标签像个书架，那这个资源字典就是一本书。

在资源字典里面新建一个附属标签，叫主题字典 *(Theme Dictionaries)* 。这个主题字典就是我们需要的东西，这里存放和主题相关的资源。

有了容器还不够，我们还需要分类，我要标明哪些资源是Light主题的，哪些是Dark主题的。

于是我们又建立了两个资源字典，分别给了一个键，标识其是Light还是Dark。

到了这一步，我们已经完成了准备工作，建了两个容器，分别是Light和Dark，软件会识别主题名字，并针对性地应用资源。

这就意味着一件事，资源必须同名。

我们最终在控件里引用的时候，只是引用一个名字，那就要求这个名字在Light中有，在Dark中也有，所以，资源必须同名。

我们新建了两个纯色颜色笔刷的资源。这个颜色笔刷你可能是第一次接触到，不要慌，后面我们还会讲，毕竟颜色在设计里是占了很大比重的。

这里我们把它当作一个普通的样式来看待，命名，设置颜色，很简单，不要纠结。

至此，我们就完成了针对不同主题的资源定制。

回到我们的MainPage.xaml。

定义一个`<Button>`如下：

```xml
<Button Background="{ThemeResource ButtonBackground}" Content="Click Me"/>
```

注意这里的Background我们使用了`ThemeResource`进行定义，想要让资源随主题而变化，则必须使用`ThemeResource`

现在。运行应用，在设置里切换主题看看吧，一切工作正常？恭喜你，完成了本节课的全部内容。

## 小结

这节课，讲了一些资源定制的事情。包括作用域以及主题资源。

值得一提的是，在课程教授中，我们直接将样式写在了标签里面。看到这句话你可能不太明白，样式不写在标签里面还能写在哪？

在实际的应用开发中，我们会创建很多的资源。这些资源如果全部写在App.xaml标签中，那么整个App.xaml会非常的臃肿。这个时候，我们往往是通过创建资源字典文件的方式，将这些资源放在不同的文件之中，然后在App.xaml里通过路径去对其进行引用。

但原理不变。

如何创建资源字典文件，如何通过路径去引用。这里面也有门道，但你可以自己去查资料。这里暂且不表。

## 思考

1. StaicResource只引用一种资源，ThemeResource则可以对主题资源进行选择。那么在ThemeResource中定义的资源可以被StaticResource引用吗？如果可以，那么引用的是哪个资源？主题变化时，资源会有什么变化？
2. 原本没有定义为主题资源的资源（比如上节课我们建立的Style），却被ThemeResource引用，会是什么结果？
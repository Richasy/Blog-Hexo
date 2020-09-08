---
type: post
title: UWP小书 - 控件模板
date: 2019/6/22 14:19
lang: zh
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：  
1. 控件的状态
2. 控件的视觉结构
3. 控件样式的修改
:::

视频地址：[BiliBili](https://www.bilibili.com/video/av56575396/)

<!--more-->

## 控件的状态

在之前的课程中我们了解到，可以通过修改属性，比如`Background`来简单地修改控件的样式。但对于有状态的控件来说，这种修改只是表象。

以`Button`为例，Button常见的有四种状态：

1. 普通状态`Normal`
2. 指针划过状态`PointerOver`
3. 按下状态`Press`
4. 禁用状态`Disable`

这几种状态的表现形式各不相同，通常反应为背景色、前景色及边框的变化。

如果想要在样式上完整客制化按钮，就需要对这几种状态进行覆写。状态覆写有多种方式，从小到大，我们可以依次来讲述。

本节课的主要内容就是帮助你实现一个按钮的完整样式自定义。

## 状态从哪来

我们刚刚以按钮为例，举了它的几种状态，但很显然，这些状态并不普适于所有控件，每种控件都有着专属状态。我们又从何得知其有哪些状态呢？

其实在最初我学习的时候，是使用`Blend for Visual Studio`。

我们可以简称其为`Blend`。

这是一个VS的扩展应用，在安装VS的时候，这个也是附赠的。从介绍上来说，它可以成为设计师的工具，但究其实质，其实是不外乎是一个偏于设计的Visaul Studio。

我至今也没有深入研究过它，所以我的表述可能有些偏颇。如果你感兴趣，可以自己尝试，这里我们仅使用VS来帮助我们获取一个完整的控件模板。

---

我们新建一个项目，并在MainPage中放置一个Button。

选择侧边栏中的`文档大纲`，也可以使用快捷键`Ctrl`+`Alt`+`T`快速呼出。

找到我们新建的Button，右键选择`编辑模板`->`编辑副本`，创建一个原始模板的副本。

![1db47924-eeb7-4d38-8afb-ba9729f2b0fd.png](https://storage.live.com/items/51816931BAB0F7A8!13412?authkey=AO7QXpgYo7-5DUU)

模板插入的地方姑且默认。默认情况下会插入到本页的Page.Resource中，这个我们不陌生了，这标志着控件模板本质上也是一个样式资源。

插入后，你会发现当前页一下子多出了好多东西，看得人眼花缭乱。不要慌，这时候千万要稳住心态，咱们可以一起来分析一下这个控件模板的结构（原始代码略长，这里就不全部贴出来了）

## 控件的视觉结构

在观察插入的控件模板时，我们可以先将`Template`标签折叠起来。

```xml
<Style x:Key="ButtonStyle1" TargetType="Button">
   <Setter Property="Background" Value="{ThemeResource ButtonBackground}"/>
   <Setter Property="Foreground" Value="{ThemeResource ButtonForeground}"/>
   <Setter Property="BorderBrush" Value="{ThemeResource ButtonBorderBrush}"/>
   <Setter Property="BorderThickness" Value="{ThemeResource ButtonBorderThemeThickness}"/>
   <Setter Property="Padding" Value="{StaticResource ButtonPadding}"/>
   <Setter Property="HorizontalAlignment" Value="Left"/>
   <Setter Property="VerticalAlignment" Value="Center"/>
   <Setter Property="FontFamily" Value="{ThemeResource ContentControlThemeFontFamily}"/>
   <Setter Property="FontWeight" Value="Normal"/>
   <Setter Property="FontSize" Value="{ThemeResource ControlContentThemeFontSize}"/>
   <Setter Property="UseSystemFocusVisuals" Value="{StaticResource UseSystemFocusVisuals}"/>
   <Setter Property="FocusVisualMargin" Value="-3"/>
   <Setter Property="Template" ...>
<Style/>
```

这一折叠，聪明的你应该看出来了。这跟我们上节课写的所谓公共样式没什么两样嘛，也就是属性多写了一些罢了。

同时我们也可以观察到，在控件模板中，一些样式属性存在着主题资源引用`ThemeResource`。如果你将光标移到这些资源名上，并按`F12`，VS将跳转到一个名为`generic.xaml`的文件中，该文件不存在于你的项目里，而是属于整个系统。

`generic.xaml`是Windows10的资源字典。

---

现在让我们打开`Template`标签。

`Button`的控件模板是比较简单明确的。我们可以在其中找到`VisualStateGroup`以及其下属的`VisualState`。这里就是按钮在不同状态下展现不同样式的奥秘所在。

这里选取`PointerOver`状态进行说明：

```xml
<VisualState x:Name="PointerOver">
    <Storyboard>
        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="ContentPresenter" Storyboard.TargetProperty="Background">
            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBackgroundPointerOver}"/>
        </ObjectAnimationUsingKeyFrames>
        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="ContentPresenter" Storyboard.TargetProperty="BorderBrush">
            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonBorderBrushPointerOver}"/>
        </ObjectAnimationUsingKeyFrames>
        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="ContentPresenter" Storyboard.TargetProperty="Foreground">
            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource ButtonForegroundPointerOver}"/>
        </ObjectAnimationUsingKeyFrames>
        <PointerUpThemeAnimation Storyboard.TargetName="ContentPresenter"/>
    </Storyboard>
</VisualState>
```

`StoryBoard`，直译过来叫故事板，但从其作用来说，我们可以称其为**动画板**，UWP的动画很多都依赖于动画板来完成。这一点在我们以后将动画的时候会说明，现在你可以简单地理解为这是一个动画容器。

`DiscreteObjectKeyFrame`，通过最后的KeyFrame我们了解到，这是一个关键帧，对视频剪辑或者动画制作有了解的同学都知道关键帧是什么意思。我们常说的帧率，其实也就是每秒展示多少帧画面。这个感兴趣的同学可以自行了解，这里提出关键帧，表示这里是一个静止的状态。

什么意思？

当你观察按钮在不同状态间的切换你会发现，按钮没有过渡动画。比如鼠标移动时按钮的边框从无到有，鼠标按下时按钮的颜色加深，这些状态的改变都是瞬间完成的，没有任何的过渡。

这也就是这里的关键帧的作用了。

回到正题，我们这节课的主要目的是为了修改按钮在不同状态下的样式。所以仔细观察，你会发现，每一个关键帧的标签上都会对应一个`TargetProperty`的附加属性。这个属性的值对应的就是所修改的`Button`的样式属性了。

接下来就是重头戏了，如何根据状态修改对应的样式属性呢？

## 控件样式的修改

不同状态下控件样式的修改，我会给大家介绍两种方法：

### 1. 覆写笔刷资源

既然状态变化所引起的样式变化本质上是笔刷资源的替换。比如从`Normal`状态变为`PointerOver`状态，**Background**的值就从`ButtonBackground`变成了`ButtonBackgroundPointerOver`。那么我们就可以通过覆写的方式，将原本的`ButtonBackground`变成我们自己定义的笔刷资源。

请在`Page.Resource`标签中加入以下代码：

```xml
<SolidColorBrush x:Key="ButtonBackground" Color="Red"/>
<SolidColorBrush x:Key="ButtonBackgroundPointerOver" Color="Blue"/>
```

这两行代码的位置在我们*ButtonStyle*的上面。

这个时候运行应用，你会发现按钮的颜色发生了改变。普通状态下是红色，鼠标移过则变成了蓝色。

这个时候你甚至可以在Button中去除对控件模板的引用，它也可以生效。

为什么？

因为我们覆写了引用的笔刷资源。

怎么覆写的？

建立一个同名的笔刷资源即可。

我们知道，默认的控件模板所引用的资源都在`generic.xaml`这个文件之中。所有UWP软件的默认资源都指向它。也正因为如此，它的优先级是最低的。只要我们在软件层级的资源列表里建立一个同名资源，那么默认的资源就会被覆盖。

这就是我们的第一种方式，覆写同名资源。

- ***优点***：很显然，相比起之前一大串控件模板，这仅仅一两行的代码无疑是简洁不少的。如果只是想改改颜色，这种方式无疑是最省事的。
- ***缺点***：覆写样式存在一个前提，那就是你必须知道你要覆写的样式的名字。这个名字并不存在于智能提示中，自己如何去找？像之前操作的那样，先建立一个控件模板副本，再去查看对应的样式名称？这有些绕弯路。但目前仍没有一个很优雅的方式解决这个问题。再者，同一页面不允许有两个同名资源，这意味着一旦你覆写了资源，在该页面中就不能再覆写了。如果我这个页面有两种按钮，需要有不同的表现形式，那么使用资源覆写的方式就很难实现我们的需求。

### 克隆模板

这种方式更为常用，也是修改控件模板时的首选方法。

还记得我们刚开始的操作吗？通过文档大纲找到对应控件，然后右键新建一个控件模板的副本。

这就完成了一个克隆的过程，我们得到了一份默认模板的拷贝。

接下来，我们需要修改的就是这个控件模板的副本，然后把它作为一个样式资源，通过Style引用即可。

这个时候，我们可以自定义笔刷资源了。

```xml
<SolidColorBrush x:Key="CustomBackground" Color="Yellow"/>
<SolidColorBrush x:Key="CustomBackgroundPointerOver" Color="Orange"/>
```

我们新建两个自定义的笔刷资源，然后在控件模板中，找到我们Background对应的关键帧，把里面的主题资源引用替换成我们自己的资源即可。

替换之后如下所示：

```xml
<Setter Property="Background" Value="{ThemeResource CustomBackground}"/>
...
<VisualState x:Name="PointerOver">
    <Storyboard>
        <ObjectAnimationUsingKeyFrames Storyboard.TargetName="ContentPresenter" Storyboard.TargetProperty="Background">
            <DiscreteObjectKeyFrame KeyTime="0" Value="{ThemeResource CustomBackgroundPointerOver}"/>
        <ObjectAnimationUsingKeyFrames/>
        ...
    </Storyboard>
</VisualState>
```

将资源引用添加到Button按钮之后，运行软件试试吧。

现在只是修改了`PointerOver`状态下的背景色。但是步骤已经说明了，现在让你去修改`Press`以及其它状态的控件样式，对你来说已经不是难事了。

这种修改控件模板副本的方式有什么好处呢？

1. 相对独立。由于所引用的主题资源可以自定义，那么这个资源就可以专门化，而非所有控件共用。这样我们就可以建立专门化的按钮，而不必担心样式资源冲突。
2. 自定义程度高。控件模板不单单是修改个颜色那么简单。默认提供给你的，是边框、背景色和前景色的属性修改，但是你完全可以仿照它们进行属性的追加，添加更多随状态改变而变化的属性。当然了，这里面也有一些坑，涉及到更为复杂的XAML语法，这里不展开来说了。
3. 可以修改按钮的表现形式。没谁规定按钮一定要是长方形的，对吗？留意Style中的`ContentPresenter`标签，它代表按钮控件的内容展示部分。你可以通过给它追加`CornerRadius`属性，配合上相等的宽高，把按钮变成一个圆形。这是完全可以做到的。

---

## 小结

本章只是初窥门径。真正要进行完整的控件模板修改，还涉及到许多其它的高级知识，比如XAML的括号式语法，动画以及其它的高级知识。可以说，掌握了控件模板的人就是半个XAML专家了。（当然，我自己也没有完全掌握）

但就实际开发而言，多数情况下也就是改改控件的颜色样式，并不会涉及更多。所以本章所讲的内容完全可以在实际开发中得到应用。

这一章虽然讲的内容不深，但却极为重要。可以说是样式修改里的核心步骤。如果你是一个有追求的软件设计者，这一章的内容是必须要掌握的，我们必须要将控件的状态变化牢牢控制在手里。

## 小尝试

1. 完整修改按钮的控件模板，逐个将不同状态的笔刷资源进行替换。
2. 在页面中，控件模板本身占了很大的版面，这并不利于我们的界面编写。结合上节课的知识，你打算如何处理这个控件模板的样式？
3. 创建一个新的控件模板副本，并将这个控件模板改造成正圆形的按钮（Width=50;Height=50;CornerRadius=25）
---
title: UWP小书 - RelativePanel 布局
lang: zh
date: 2019/5/4 20:45:12
type: post
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- RelativePanel 的相对方位
- RelativePanel 的适用场景
:::

## 什么是相对方位

在日常生活中，当我们想描述某个物体的位置时，我们一般都会用相对位置来描述。比如“小张就在班主任的后面”，而不是“小张在北纬42°，东经112°”。

相对位置的描述总是会先确定一个参照物，然后其它物体的位置信息都是相对参照物而言的。而这一点，正是 **RelativePanel** 的布局核心。

<!--more-->

## RelativePanel 如何布局

上节课，我们讲述了StackPanel ，那时候我们说StackPanel内的子元素像排队，秩序井然，排序严格。但在实际布局中，我们并不总是需要这种严格的布局排版。

就如上节课举的例子，当一个容器内只有两个元素，我想让它们一左一右地排列，StackPanel 就不能实现，但 RelativePanel 却可以。因为在 RelativePanel 中，并不会对子元素的位置进行限制，其中的子元素理论上可以出现在容器内的任何位置。

RelativePanel 又名**相对布局**，在RelativePanel 中，确定子元素位置的除了常规的`Margin`之外，便是一系列用以描述相对位置的属性，概览如下：

|属性名|说明|
|-|-|
|Above|在某元素上方|
|Below|在某元素下方|
|LeftOf|在某元素左边|
|RightOf|在某元素右边|
|AlignTopWith|本元素的上边缘与某元素的上边缘对齐|
|AlignBottomWith|本元素的下边缘与某元素的下边缘对齐|
|AlignLeftWith|本元素的左边缘与某元素的左边缘对齐|
|AlignRightWith|本元素的右边缘与某元素的右边缘对齐|
|AlignVerticalCenterWith|本元素与某元素的垂直居中对齐|
|AlignHorizontalCenterWith|本元素与某元素水平居中对齐|
|AlignTopWithPanel|是否与容器上边缘对齐|
|AlignBottomWithPanel|是否与容器下边缘对齐|
|AlignLeftWithPanel|是否与容器左边缘对齐|
|AlignRightWithPanel|是否与容器右边缘对齐|
|AlignVerticalCenterWithPanel|是否在容器中垂直居中对齐|
|AlignHorizontalCenterWithPanel|是否在容器中水平居中对齐|

通过上面的表格你就可以发现，RelativePanel 为子元素提供了大量的附加属性来帮助确定其相对位置 *（其中的垂直居中前端开发者看了想必会很感动）*  。而正如本文开头所述，相对位置建立在至少有一个参照物的基础上。

我们可以说“Logo在导航按钮的右边”，那么这个时候我们就要在Logo所代表的元素标签内使用附加属性`RelativePanel.RightOf="导航按钮"`，这样你也发现了，为了能获取导航按钮元素，那么就必须事先给导航按钮命名，也就是给导航按钮的`Name`属性赋值。

话不多说，让我们来实现一个简单的顶部导航栏布局吧。

## 顶部导航栏布局

在本例中，我们需要四个元素，分别是菜单按钮、标题文本、后退按钮和前进按钮。最终布局如下图所示：

![uwp_relativepanel_01.png](https://storage.live.com/items/51816931BAB0F7A8!12611?authkey=AO7QXpgYo7-5DUU)

首先我们为顶部导航栏腾出空间来，即将整个页面分作`导航栏`+`内容区`。这里我们就可以利用Grid进行操作：

```xml
<Grid>
     <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
     </Grid.RowDefinitions>
</Grid>
```

第二行不管，我们在第一行新建一个 RelativePanel。

```xml
<Grid>
     <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
     </Grid.RowDefinitions>

     <RelativePanel Height="50" Background="DarkGray">

     </RelativePanel> 

</Grid>
```

在此之后，我们先把所用到的元素写进去

```xml
<Grid>
     <Grid.RowDefinitions>
         <RowDefinition Height="Auto"/>
         <RowDefinition Height="*"/>
     </Grid.RowDefinitions>
     <RelativePanel Height="50" Background="DarkGray">
         <Button Width="50" Height="50" Content="&#xE700;" FontFamily="{StaticResource SymbolThemeFontFamily}" Name="MenuButton" />
         <TextBlock Name="HeaderTextBlock" Text="首页" FontSize="20" FontWeight="Bold" />
         <Button Name="NextButton" FontFamily="{StaticResource SymbolThemeFontFamily}" Content="&#xE72A;"  Height="50" Width="50" />
         <Button Name="PreviousButton" FontFamily="{StaticResource SymbolThemeFontFamily}" Content="&#xE72B;"  Height="50" Width="50" />
     </RelativePanel>
</Grid>
```

到了这里，我们就完成了一半的工作了。接下来的一半则是本章内容的核心，即给元素确定位置：

1. 菜单按钮在页面最左边，故使用与容器相对齐的`AlignLeftWithPanel`属性。
2. 标题文本在菜单按钮的右边，并与其有一定的间隔，同时它又是垂直居中的，所以要搭配使用`RightOf`、`AlignVertialCenterWithPanel`和`Margin`。
3. 前进和后退按钮在页面的右侧，而前进按钮在最右侧，故优先写前进按钮，之后后退按钮便以前进按钮为参照进行定位，涉及属性`AlignRightWithPanel`和`LeftOf`。

故完整代码如下：

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/>
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <RelativePanel Height="50" Background="DarkGray">
        <Button Width="50" Height="50" Content="&#xE700;" FontFamily="{StaticResource SymbolThemeFontFamily}" Name="MenuButton" RelativePanel.AlignLeftWithPanel="True"/>
        <TextBlock Text="首页" FontSize="20" FontWeight="Bold" RelativePanel.RightOf="MenuButton" RelativePanel.AlignVerticalCenterWithPanel="True" Margin="15,0,0,0"/>
        <Button Name="NextButton" Content="&#xE72A;" FontFamily="{StaticResource SymbolThemeFontFamily}" Height="50" Width="50" RelativePanel.AlignRightWithPanel="True"/>
        <Button Name="PreviousButton" Content="&#xE72B;" FontFamily="{StaticResource SymbolThemeFontFamily}" Height="50" Width="50" RelativePanel.LeftOf="NextButton"/>
    </RelativePanel>
    </Grid>
```

## RelativePanel 的适用场景

如你所见，RelativePanel 的布局更加自由，同时又能通过相对位置保留一丝“秩序”<small>*这个秩序是相对完全自由的绝对定位容器Canvas而言的*</small>。

RelativePanel 常用于卡片式布局之中。比如一个视频卡片，卡片的高度可能随着内容会有变化，但始终要求视频发布日期（或标签、点赞数等）在卡片底部。类似这种场景就可以考虑 RelativePanel 。

其它的还有很多，比如对居中对齐有要求的布局，都可以使用RelativePanel。

RelativePanel虽有其局限，但同时也是非常强大的布局工具，尤其是在强调自适应的UWP应用中，更是有着得天独厚的优势，它能够确保在软件窗口大小改变时，布局仍能维持大体稳定。

## 小结

本章对相对布局容器`RelativePanel`有了一个大概的介绍。RelativePanel 会给子元素提供很多的附加属性用以描述其相对位置，这个相对位置的参照可以是容器，也可以是容器内的其它子元素。

而落实到生产环境，Grid、StackPanel、RelativePanel，这些布局控件的特性都有一定重叠，换句话说，总会有替代方案，只不过有些容器控件会更容易达到目标罢了，了解这些布局容器的特性，按照自己的需要灵活选取合适的容器才是你真正要做的。

除了介绍的Grid、StackPanel和RelativePanel之外，还有如Canvas、VariableSizedWrapGrid等布局控件（在Windows Community Toolkit中还有更多），这些控件我不再细说，如果你感兴趣，可以自行了解。

## 延伸阅读

- [布局面板](https://docs.microsoft.com/zh-cn/windows/uwp/design/layout/layout-panels)
- [Windows Community Toolkit](https://github.com/windows-toolkit/WindowsCommunityToolkit)

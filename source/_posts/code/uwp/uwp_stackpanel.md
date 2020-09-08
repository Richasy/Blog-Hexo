---
title: UWP小书 - StackPanel 布局
lang: zh
date: 2019/5/4 17:45:12
type: post
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- StackPanel布局方式
- StackPanel排版规则
:::

> 视频地址：[BiliBili](https://www.bilibili.com/video/av49066072/)

<!--more-->

## StackPanel是什么？

在之前讲文本排版时，我曾告诉大家可以使用StackPanel来让两个文本变成上下排列，不至于重叠。

在那个时候，细心的你就会发现StackPanel异于Grid的地方。

StackPanel 又被称作堆放布局。它会把其中的子元素按着先来后到的顺序排成一列（或一行），没有人插队，没有人并列，更不会有人重叠。

这样的一种布局控件在实际开发中用得是非常多的，它就好比HTML里的`<div>`，属于最常见的容器之一。

## 如何使用StackPanel

StackPanel能帮助你完成两种排版，水平(`Horizontal`)方向和垂直(`Vertical`)方向的元素堆叠。

控制StackPanel排布方向的属性名为`Orientation`，它可以设置为`Horizontal`或`Vertical`，分别对应水平排布和垂直排布。默认的排布方向则是垂直排布。

垂直方向上是从上至下，水平方向上是从左至右。与我们的阅读习惯一致。

现在我们新建一个项目，将`MainPage.xaml`中的Grid换成StackPanel，并输入以下代码：

```xml
<StackPanel>
    <Button Content="列表项1"/>
    <Button Content="列表项2"/>
    <Button Content="列表项3"/>
    <Button Content="列表项4"/>
    <Button Content="列表项5"/>
    <Button Content="列表项6"/>
</StackPanel>
```

现在你可以看到在预览界面上出现了6个堆在一起的按钮，它们的顺序和你写在XAML里的顺序一致。

![uwp_stackpanel_01.png](https://storage.live.com/items/51816931BAB0F7A8!12563?authkey=AO7QXpgYo7-5DUU)

现在你可以试试将StackPanel的`Orientation`改为`Horizontal`，看看会发生些什么？

## StackPanel的用武之地

在UWP的布局中，我们常常会接触到一些重复元素，比如视频列表里每一个视频卡片，尽管它们的内容不尽相同，但布局却都是一致的，这一个个布局相同的元素拼在一起就组成了整个界面，所以我们可以称每个卡片为一个单元，StackPanel就很适合作为单元容器。

说直白一些，你可以将一些重用性高的布局组合丢进一个StackPanel里，然后你就可以将之作为一个整体进行控制。

举个例子，一个简单的图标+文本的排版，这种排版方式随处可见，哪怕我不放图片你也可以脑部一堆现成的例子（比如你的手机设置列表）

想一想，如果你在每一处用的地方都将图片控件和文本控件单独写一次，繁琐不说，还极有可能对周围的控件产生干扰，而将这两者放入StackPanel，就相当于给它们了一个小窝。在设置Margin等属性时，就可以对这个整体进行操作，提高了布局的稳定性。

```xml
<StackPanel Orientation="Horizontal">
    <Image Source="图片路径" />
    <TextBlock Text="内容" />
</StackPanel>
```

也有一种有意思的用法，即将StackPanel用作分割线。

提起线，大多数人想到的都是Line，这是线条的英文单词，而在UWP中，也的确存在Line这个控件，它就是用来画线的，但遗憾的是，它是Shape类型的控件。

Shape类型的控件有一个特点，它们必须要指定明确的长/宽，或者诸如此类的属性，通过这些属性定义出的Shape当然不会支持自适应。

在UWP里你想要画一条线，需要指定这条线的起点与终点坐标，即X1, Y1和X2, Y2的值。这种控件拿来做界面的分割线显然就不太合适了，尤其是在你不知道容器有多宽的情况下。

但我们可以利用容器控件的自适应特性，将其`Width`/`Height`设置为你想要的线条粗细，再将其另一个方向上的Alignment设置为`Stretch`，再把`Background`的颜色补上，就可以得到一条自适应的垂直线或水平线了。

---

StackPanel的一大优势就在于逻辑清晰，操作方便，你将子元素放进去时，明确地知道它会怎么排版。不必像Grid一般先定义行和列，所以它特别适合做一些简单的布局，省去很多预设的功夫。

但StackPanel同样有自己的局限。

## StackPanel的一些小坑

默认情况下，StackPanel的面积是由其内部元素所占的面积决定的，但这一点可以通过将HorizontalAlignment或VerticalAlignment设置为`Stretch`来改变，设置之后，StackPanel会占据当前水平/垂直方向剩余的空间。

但这并不是一定成立的，比如在ListView之中。

关于ListView，这是我们后面的内容。这里你只需要知道，当你在ListView中进行子项模板的编写时使用了StackPanel，即便你将`HorizontalAlignment`设置为`Stretch`也不会让当前StackPanel占据水平方向的全部空间。

所以说，凡事总有例外。

同时还有一些在Grid中很正常的操作在StackPanel中就会出现问题。比如子元素的水平位置。

在Grid中，我们可以通过HorizontalAlignment来决定当前子元素的水平位置，定位准确，执行得一丝不苟，但在StackPanel中不是这样。

你可以试试如下代码：

```xml
<StackPanel>
     <StackPanel Orientation="Horizontal" HorizontalAlignment="Stretch" Background="LightGray">
        <Button Content="列表项1" HorizontalAlignment="Left"/>
        <TextBlock Text="测试文本" HorizontalAlignment="Right"/>
    </StackPanel>
</StackPanel>
```

按照我们的预期，StackPanel占据了水平方向的全部宽度，按钮与文本控件应该分列左右两端，但事实上并非如此。

在预览中可以看到，StackPanel的确占据了水平方向的全部宽度，但文本控件仍然死死粘着按钮不肯动弹。

![uwp_stackpanel_02.png](https://storage.live.com/items/51816931BAB0F7A8!12564?authkey=AO7QXpgYo7-5DUU)

这是为什么呢？

因为我们的操作违反了StackPanel的排版规则。

在之前的堆叠排版中我们可以看出，StackPanel内部的元素都是规规矩矩地“排队”，而且有着极其明确的先来后到的观念，因为它们到了StackPanel的地盘就要遵循它的规则。

**排队**，正是StackPanel的规则。

回头看看我们的代码，我们干了什么？

让一个排行老二的人滚去当吊车尾（水平方向上）？

难怪它不愿意了。

试想这时候如果在TextBlock的后面还有第三个元素，假设TextBlock听从我们的意愿跑到了右边，这第三个元素怎么办，顺位补上吗？显然不可能。

这个时候我们就可以得出一个结论，StackPanel在水平排布时，子元素的`HorizontalAlignment`属性无效；在垂直排布时，子元素的`VerticalAlignment`属性无效。

所以我们在StackPanel内进行布局的时候，一定要注意其内部的排列规则，不要发布“乱命”。

## 小结

StackPanel相对于Grid来说使用更简便，但其内部规则相对来说更为“死板”，其中子元素严格遵循既定的规则排布，这并非坏事，事实上，这反而能减少工作量，能提供一种确定的排版方式，这对设计者来说是难能可贵的（来自前端的怨念）。

但是对于那些追求自由的创作者们，StackPanel过于死板的排布规则很难满足它们旺盛的创作欲，但Grid的配置又略有些繁琐（你可以试想用Grid实现10张图片的阶梯状排列），这时候，UWP给我们提供了另一种布局工具：RelativePanel。当然，这个是后话了。

## 延伸阅读

- [采用 XAML 的响应式布局](https://docs.microsoft.com/zh-cn/windows/uwp/design/layout/layouts-with-xaml)
- [布局面板-StackPanel](https://docs.microsoft.com/zh-cn/windows/uwp/design/layout/layout-panels#stackpanel)

## 小尝试

在上节课讲 Grid 时，我们尝试实现了一个很赞的文本排版，那时我们全是用Grid来实现的，现在你可以尝试将其中能用StackPanel替换的地方替换一下。
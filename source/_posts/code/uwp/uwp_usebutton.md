---
title: UWP小书 - 一个小按钮
lang: zh
date: 2019/5/3 21:45:12
type: post
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- Button控件的使用
- 创建Click事件
- 如何弹窗
- 通过C#改变控件属性
:::

视频地址：[BiliBili](https://www.bilibili.com/video/av47338535/)

按钮这个东西跟文本控件一样，是我们平时最常见到的控件之一。现在我们看到按钮都会有一种潜意识，只要点击了按钮，就会有一些神奇的事情发生。

本章，我们将建立一个按钮控件，亲手来做一些神奇的事情。

上节课写下的代码你还没有丢到回收站里吧。这节课我们将利用上节课写好的`Hello World`来做一些事。

<!--more-->

## 建立一个按钮

将`Grid`替换为`StackPanel`，并写一个`Button`控件：

```xml
<StackPanel VerticalAlignment="Center">
    <TextBlock FontWeight="Bold" TextAlignment="Center" FontSize="40" HorizontalAlignment="Center">Hello World</TextBlock>
    <Button HorizontalAlignment="Center">点我</Button>
</StackPanel>
```

我们可以在Button标签内写上“点我”，这个`点我`最终会作为Button按钮的文本出现。

运行一下应用看看，你能看到在`Hello World`的下方出现了一个上面写着*点我*的按钮。当然，你现在点击按钮是没有任何反应的 ，因为你还没告诉它它要做什么。

## 创建点击事件

回想一下这样的场景，主角一行到某个墓里探险，一路上碰到了许多的机关，险象环生。这些机关之所以生效，要么是有人踩了某个砖头，要么是摸到了一些奇怪的装饰，总而言之，是满足了机关启动的条件，所以机关被触发。

咱们的按钮也是一个机关，而想要让它启动，便要给它一个启动的条件。而按钮最常见的启动条件就是点击。

在应用开发中，我们称这样的启动条件为事件（Event）,而点击事件，被称作`Click`。

现在，在Button控件中敲下`Click`，VS会提示你*新建事件处理程序*，回车确认，这样你就为按钮新建了一个点击事件了。

![uwp_usebutton_01.png](https://storage.live.com/items/51816931BAB0F7A8!12447?authkey=AO7QXpgYo7-5DUU)

## 告诉按钮该怎么做

光有了事件还不行，如果你是古墓的设计者，有人踩了你设计好的砖头，你总要有后手，放冷箭也好，地面塌陷也罢，总之要设定踩了砖头之后会发生什么。

应用也是同样，给Button新建Click事件之后，我们要告诉软件，我按了按钮之后你要做什么事。

如果你没啥想做的，那么我给你个建议，你可以点击按钮之后来个弹窗，上面写`云之幻最帅！`😀。

接下来我们来看看如何做。

还记得上节课我们提到的Code-Behind文件吗？点开MainPage.xaml前面的三角，你会发现里面还藏着一个MainPage.xaml.cs，这里就放着我们的C#代码了。

双击打开，拉到最下面，你会看到你刚刚创建好的事件（事实上，通过将光标定位在xaml的事件上，按F12也可以直接跳转到该事件的代码上）。

==对于怎么写C#，我在最开始就说过，我这里是UWP教程，并不负责教C#，所以我希望你对你眼前的这些代码并不会一头雾水。==

既然要做弹窗，我们就需要知道UWP为我们提供的弹窗控件是什么。

UWP提供的弹窗控件主要有两个，一个是`MessageDialog`，一个是`ContentDialog`，前者比较简单，我们可以直接应用。

在事件中写入如下代码：

```csharp
private async void Button_Click(object sender, RoutedEventArgs e)
{
    // 创建MessageDialog
    MessageDialog tipDialog = new MessageDialog("云之幻最帅！");
    // 异步显示对话框
    await tipDialog.ShowAsync();
}
```

*在写代码的时候，如果MessageDialog下出现波浪线，并且没有高亮，把鼠标移上去，点击灯泡，引入对应的包即可。*

**现在，运行一下软件，点击按钮，看看是不是弹窗了？**

*关于Async/Await在UWP中的应用解释，请看[小结](#小结)，并且在教程中期会重点讲解异步*

## 通过代码修改文本

反正已经建了一个按钮，不妨多玩点操作。

方才我们通过点击按钮，弹出了一个对话框。现在我们不弹对话框了，我们来修改文本。

现在软件中央显示着一个大大的Hello World，我要求你点击按钮之后，文本控件中的Hello World变成`你好，世界`。

听上去好像不难，如果你有这种想法，可以自己先尝试一下，看看怎么解决。

---

这波操作有两个问题：

1. 我该怎么在C#代码中操作XAML里定义的控件？
2. 我该怎么修改TextBlock的文本？

回顾一下你学习C#的时候，你想使用一个变量先要干什么？先定义变量。换句话说，你要先给变量取个名字，才能在之后的代码中调用，正如我们之前先定义了一个`tipDialog`，之后才能调用`tipDialog`的`ShowAsync`方法。

而所有XAML中的控件，本质上和MessageDialog一样，都是预设的**类**！（敲重点）

现在，你需要拿到在XAML中定义的`TextBlock`，就要先给它取个名字才行。让我们回到MainPage.xaml，修改TextBlock的`Name`属性：

```xml
<TextBlock ... Name="MyTextBlock" ...>
```

现在回到C#页面，输入MyTextBlock，看看发生了什么，你还没输完，VS就已经给你提示了！

这意味着你已经可以拿到名为MyTextBlock的TextBlock控件进行操作咯。

解决了这个问题，还剩下一个问题等待我们解决。

在xaml中，我们直接将文本写在了两个标签的中间，在C#中可没有标签，我们怎么处理？

还记得我说过的话吗？所有的控件本质上都是**类**，既然如此，TextBlock所显示的文本也必然只是它的一个属性而已。这个属性就是`Text`。

让我们注释掉之前`tipDialog`相关的代码，并写下如下代码（记得删掉`async`关键字）：

```csharp
private void Button_Click(object sender, RoutedEventArgs e)
{
    //// 创建MessageDialog
    //MessageDialog tipDialog = new MessageDialog("云之幻最帅！");
    //// 异步显示对话框
    //await tipDialog.ShowAsync();

    MyTextBlock.Text = "你好，世界";
}
```

现在运行软件，点击按钮，看看发生了什么。

## 小结

本章带你创建了一个Button控件，了解了如何创建一个事件，并如何利用这个事件执行一些操作。你还知道了怎么在C#中获取xaml里定义的控件，也知道了如何通过C#代码来修改控件的属性。

### 关于async/await

async/await是一对关键字，焦不离孟，孟不离焦。它们就是鼎鼎大名的异步关键字，也是C#为人称道的一个语言特性。

说起异步，这个在UWP应用中用得可是太多了。可以说，UWP应用大多数的卡顿都是由于没有合理运用异步造成的。异步用好了，可以极大提高软件的交互体验。这一点我会在后期讲解，这里只简单地说明一下异步的作用。

简而言之，异步就好比在高速公路上加了个应急车道。试想，如果高速路上没有应急车道，一旦有车抛锚，高速路必然堵塞。换到应用里，如果有某个操作耗时很长（比如网速不好的时候去获取网络数据），应用的全部精力（线程）都去处理这件事了，UI就没人管了，用户就会发现软件失去响应了，卡在那里半天不动，数据获取完了才恢复响应，我想你肯定遇到过这种情况，是不是很想砸键盘？这就是没有应用异步的后果。

所以，异步就是把耗时很长的任务再开一个线程单独处理，UI继续保持响应，事儿办完了，派个人通知一下应用，然后应用再让你归队，继续完成接下来的任务。

文中所写的MessageDialog应用了异步处理，对我们而言，MessageDialog感觉是秒开，似乎用不着异步，但事实上，它创建了一个控件，对布局造成了影响，内部的开销是不小的，但凡是这种**可能**执行时间较长的操作，微软都将其强制设置为了异步方法。

## 延伸阅读

- [异步编程](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/async/)
- [UWP-对话框](https://docs.microsoft.com/zh-cn/windows/uwp/design/controls-and-patterns/dialogs-and-flyouts/dialogs)
- [UWP-按钮](https://docs.microsoft.com/zh-cn/windows/uwp/design/controls-and-patterns/buttons)

## 小尝试

恭喜你，你会使用按钮了，那么要不要尝试一些其它的操作呢？

1. 点击按钮，如果TextBlock中的文本是`Hello World`，就变成`你好，世界`，如果是`你好，世界`，就变成`Hello World`。
2. 根据[延伸阅读](#延伸阅读)中有关对话框的内容，尝试一个ContentDialog并修改主要按钮的文本
3. 根据[延伸阅读](#延伸阅读)中有关按钮的内容，尝试让按钮文本变成下图所示的样子：

![uwp_usebutton_02.png](https://storage.live.com/items/51816931BAB0F7A8!12448?authkey=AO7QXpgYo7-5DUU)
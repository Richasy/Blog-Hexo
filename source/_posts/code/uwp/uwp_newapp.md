---
title: UWP小书 - 建立第一个应用
lang: zh
date: 2019/5/2 19:45:12
type: post
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- 如何建立项目
- 如何启动项目
- 如何加入控件
- 如何修改控件属性，改变外观
:::

> 视频地址：[BiliBili](https://www.bilibili.com/video/av47065493/)

让我们先无视掉复杂的基础知识，直接开始建立第一个UWP应用吧！

<!--more-->

## 新建项目

打开Visual Studio，按以下步骤操作：

`文件 --> 新建 -->项目`，或者直接按快捷键 `Ctrl`+`Shift`+`N`。

按照下图所示选择 `空白应用(通用Windows)`，填写合适的名称并选择合适的路径，点击确定。

![uwp_first_app_01.png](https://storage.live.com/items/51816931BAB0F7A8!12439?authkey=AO7QXpgYo7-5DUU)

这时，IDE会弹出一个窗口，询问你应用支持的版本

![uwp_first_app_02.png](https://storage.live.com/items/51816931BAB0F7A8!12440?authkey=AO7QXpgYo7-5DUU)

咱们这时候就不管什么平台兼容性了，直接把最低和目标版本都设为最新的系统版本（当然，这要你的系统本身是最新版本才行）

现在，你就新建了一个UWP应用啦！

![uwp_first_app_03.png](https://storage.live.com/items/51816931BAB0F7A8!12441?authkey=AO7QXpgYo7-5DUU)

:::tip
由于我们下节课也会围绕这个新建立的APP展开，所以请认真对待哟~  
会使用Git的童鞋可以新建Git存储库，通过Git记录自己的学习过程
:::

## 启动项目

当项目建立完成之后，你会发现项目内包括了很多的文件。

这些文件具体是什么意思，且让我按下不表，因为现在的你恐怕没耐心听我解释。

让我们直接启动这个新建的应用吧！

![uwp_first_app_04.png](https://storage.live.com/items/51816931BAB0F7A8!12442?authkey=AO7QXpgYo7-5DUU)

看到VS顶部的这块区域了吗？那个绿色的形似播放按钮的三角符号就是启动按钮了。

按下，启动！

![uwp_first_app_05.png](https://storage.live.com/items/51816931BAB0F7A8!12443?authkey=AO7QXpgYo7-5DUU)

你可以看到VS下方的`输出`窗口在不停地吐字，这表明VS正在编译代码，并将编译后的应用部署在你的计算机上，部署完成后，应用会自动打开。

当然，现在的应用什么都没有，但这是个好的开始。

接下来关掉应用，让我们往应用里面塞点东西吧！

## 写下 Hello World

`Hello World`是一个颇具魔力的短语，几乎所有的编程语言都会从这里开始，这已经演变成编程传统了。那么在这里，我们便依从传统，从让应用显示`Hello World`开始吧！

### MainPage

看看我们的项目文件，找到`MainPage.xaml`，双击打开它。

![uwp_first_app_06.png](https://storage.live.com/items/51816931BAB0F7A8!12444?authkey=AO7QXpgYo7-5DUU)

现在VS中央会出现一个对半分的界面，上面是预览，下面是代码，这里就是我们写界面的地方了。通常情况下，我们是不会让这两个窗口同时存在的，这太占地方了！

在我向你解释为什么打开MainPage之前，先在两个窗口的隔离带的右侧找到向下的箭头按钮，鼠标移上去会显示**折叠窗格**，点一下，是不是感觉清爽了许多？

在界面下方有两个选项卡，你可以在XAML和设计界面之间切换。现在让我们切换到XAML。

好了，让我来解释下为什么要打开这个MainPage吧。

简而言之，MainPage就是你方才打开应用时看到的那个白白的(或者黑黑的)界面，MainPage直译过来也就是主页的意思。我们接下来敲`Hello World`就要在这个主页上敲，所以我们才要打开它。

至于为什么UWP应用启动就会显示MainPage页面，以及UWP应用能不能有其他页面的问题，我会在文末小结中做出解释。

### TextBlock

XAML说穿了，就是XML的一种变体，这是一种标记语言，这里我不会解释太多，感兴趣的同学可以在[延伸阅读](#延伸阅读)中找寻资料。

同样是XML的变体，网页所使用的HTML就比XAML名气响的多。如果你有学过网页，那么看到这个XAML的语法想必不会陌生。

相比起HTML，XAML的语法可是严格的多。

我们先忽略`Page`后面跟着的一大堆属性，把目光锁定在`Grid`上面：

```xml
<Page
    ...>

    <Grid>
        <!--这里是我们写内容的地方-->
    </Grid>
</Page>
```

`Grid`就是我们写`Hello World`的地方了，你可以试试直接在`Grid`里面写`Hello World`。

打波浪线了，对不对😀

这就是报错，告诉你不能这么写，而HTML里就可以，这也是XAML比HTML严格的一种体现。

我们想要显示文字，那么这里我们就必须要放一个显示文字的控件才行。

这个控件就是`TextBlock`

在Grid中输入以下内容：

```xml
<Grid>
    <TextBlock></TextBlock>
</Grid>
```

`TextBlock`，这是你亲手输入的第一个控件，也是你以后会接触得最多的控件，它的作用就是显示一段文本。

现在，在`TextBlock`两个标签的中间输入内容：

```xml
<Grid>
    <TextBlock>Hello World</TextBlock>
</Grid>
```

OK，没有报错，一切都很好。现在，再次运行应用。

应用的左上角，一个小小的Hello World就出现咯~

## 变粗，变大，还要居中

这样一个小小的Hello World肯定不能满足你，作为你的第一个应用里的第一句话，一定要够大够粗够醒目才行。

接下来让我们对这个`TextBlock`进行改动吧！

### 变粗

TextBlock作为一个文本控件，自然有法子控制文本该如何显示，它自带了诸多属性，使用者可以通过改变这些属性来对文本的显示方式进行调整。

让我们先把文本变粗吧。

TextBlock控制文本粗细的属性是`FontWeight`，在TxetBlock的标签内敲下FontWeight，得益于IDE的强力支持，你可以看到不用你全部输入完成，编辑器会帮助你进行自动补全，敲下`FontWeight`后，编辑器会提示你所有的候选值：

![uwp_first_app_07.png](https://storage.live.com/items/51816931BAB0F7A8!12445?authkey=AO7QXpgYo7-5DUU)

既然是加粗，我们就选择 `Bold`。

```xml
<Grid>
    <TextBlock FontWeight="Bold">Hello World</TextBlock>
</Grid>
```

运行软件看看，是不是变粗了？

### 变大

控制文本字号的属性是`FontSize`，FontSize需要设置为数字，这个数字有一个隐藏的单位，称之为`epx`，关于`epx`的具体信息，可通过[延伸阅读](#延伸阅读)查看。

UWP默认的文本字号为16，这里我们可以设置为40。

```xml
<Grid>
     <TextBlock FontWeight="Bold" FontSize="40">Hello World</TextBlock>
</Grid>
```

运行软件看看，是不是变大了呀~

### 居中

`Hello World`变得又大又粗了，但还有个缺憾，它似乎太靠左上了。这样一个大家伙，不居中不能给人以冲击感啊。

如何居中呢？

TextBlock有两个属性`HorizontalAlignment`和`VerticalAlignment`，顾名思义，前者控制水平对齐，后者控制垂直对齐。

经过前面的变大变粗，相信这里不用我赘述了。分别将这两者设置为`Center`，运行应用，看看效果吧！

```xml
<Grid>
    <TextBlock FontWeight="Bold" FontSize="40" HorizontalAlignment="Center" VerticalAlignment="Center">Hello World</TextBlock>
</Grid>
```

---

怎么样，又大又粗的`Hello World`居中显示在你的面前，这都是你自己敲出来的哦。

现在我可以恭喜你——你亲手创建了一个UWP应用！

## 小结

在创建应用的喜悦之后，让我们来沉淀一下，请你耐心地看完我接下来说的知识，这对你理解UWP应用有一定的帮助。

### 为什么软件打开之后会显示MainPage

在项目中，除了MainPage.xaml之外，还有一个App.xaml，它控制了整个应用的生命周期，换句话说，它控制了应用从打开到关闭的全过程。但你双击打开它，你会发现里面好像什么都没有。

这里就有一个玄机了。你会发现，App.xaml的前面有一个三角符号，点一下它，App.xaml的下面多出来一个文件，名为App.xaml.cs（这种隐藏式文件被称作 Code-Behind），双击打开它。

这里的信息量就很大啦，微软贴心地在里面写上了注释。细心观察，你会发现，在`OnLaunched`函数里，软件的Frame默认就导航到了`MainPage`中。所以你才会在打开软件的时候看到MainPage。

### 什么是Frame

我们可以类比浏览器。

浏览器的结构是什么样的呀？上面是控制面板，下面是内容浏览。

那个内容浏览的区域，就是Frame。整个UWP应用，就好比一个去掉控制面板的浏览器。

你的MainPage就好比是一个网页，网页可以在浏览器里来回跳转，可以切换到其它的网页，只要在Frame里，那就随你折腾，所以有些人戏称UWP就是个浏览器套壳应用。说是这么说，各位可一笑置之，莫要当真。

由此可知，一个UWP应用可以有很多个Page，正是通过这些Page之间的切换，我们才可以实现应用的诸多功能。但一个应用是不是也可以有很多个Frame呢？

是的，可以有！

但RootFrame只有一个，那就是Window。从视觉上说，就是你的应用周围那浅浅的灰边组成的框。Frame里可以嵌套Frame，但大家的祖宗都是Window。这就是UWP应用区别于传统Windows应用的一个特点。

过去的桌面应用，基本组成单元就是Window，软件是Window，对话框也是Window，只要你想，一个Win32应用可以弹出很多个Window，但UWP应用却只有一个Window（不久之后将允许多实例，但本质未变）。

### TextBlock为什么要写在Grid里面？

问出这句话，相信你已经尝试把TextBlock写在外面了，勇于尝试，这是好事。尝试过之后，你会发现即便不要Grid也可以显示文本。但如果想再加一个TextBlock，IDE就会无情报错，告诉你`已多次设置属性Content`。

这里的Content，指的是`Page`的属性Content，在两个Page标签之间的内容，均可视作`Content`。

Page的`Content`属性只允许其中有一个元素，换句话说，里面只能有一个TextBlock。而我们的`Grid`，它里面允许有多个元素，这就是我们为什么要把TextBlock放在Grid中的原因。

像`Grid`这样可以容纳其它控件的控件，我们称之为容器，类似的容器还有`StackPanel`、`RelativePanel`等。

## 延伸阅读

- [XML介绍](http://www.runoob.com/xml/xml-intro.html)
- [XAML教程](https://www.tutorialspoint.com/xaml/)
- [UWP 应用设计简介-有效像素和缩放](https://docs.microsoft.com/zh-cn/windows/uwp/design/basics/design-and-ui-intro#effective-pixels-and-scaling)
- [官方教程-创建Hello World应用](https://docs.microsoft.com/zh-cn/windows/uwp/get-started/create-a-hello-world-app-xaml-universal)

## 小尝试

完成上面的应用创建其实很简单，并不会花多长时间，但如果你愿意挑战一下自己的话，我也为你准备了一些你能完成的操作，但可能需要你稍微查一下资料。

1. 更改文本的字体（FontFamily）
2. 更改文本的对齐方式（TextAlignment）
3. 更改文本颜色（Foreground）
4. 试试改变TextBlock的位置，设置为左上，右上，右下，左下
5. 新建一个TextBlock控件，写上很多字，形成如下排版：
    ![uwp_first_app_08.png](https://storage.live.com/items/51816931BAB0F7A8!12446?authkey=AO7QXpgYo7-5DUU)
    - Grid造成文本重叠的话，试试StackPanel
    - 文本不能自动换行的话，试试`MaxWidth`和`TextWrapping`的组合
    - 用用`Margin`






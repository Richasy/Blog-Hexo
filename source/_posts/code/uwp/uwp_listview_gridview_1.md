---
title: UWP小书 - ListView 和 GridView (上)
lang: zh
type: post
date: 2019/11/17 10:00:00
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- ListView/GridView 的用法
- 与集合的绑定
- ObservableCollection的用法
:::

通过本章的学习，你将接触UWP最重要的集合控件：ListView和GridView。它们虽然是两种控件，但原理相通，最大的区别也不过在于排版方式的不同，故而将这两者合在一起讲。

ListView和GridView控件本身并不难，不足以让我分成上下两章来说，但其背后的数据绑定、数据模板等，却是制作UWP应用时必须要具备的知识，值得大书特书。

<!--more-->


## 从 ListView 开始

为什么说ListView和GridView是最重要的集合控件？

因为几乎所有的软件都属于“信息展示”应用，而信息展示应用对于相同数据结构的展示大多都依赖于集合控件。

举个例子，QQ的聊天列表。

![d61eb78d-3a26-4944-b8bc-5f2a76d2c09d.png](https://storage.live.com/items/51816931BAB0F7A8!13558?authkey=AO7QXpgYo7-5DUU "图片来自Dribbble")

这就是典型的ListView。

我有一个数据集合，比如`List<string>`，通过ListView，我只需要创建单一条目的UI样式，ListView就会根据集合将所有条目的UI都创建好。

所以你可以看到，每条聊天记录的头像、名字、信息等都不相同，但布局一致，这就是ListView的能力。

现在，我们可以先来简单地试用一下`ListView`。

```xml
<Grid>
    <ListView>
        <ListViewItem Content="First Item"/>
        <ListViewItem Content="Second Item"/>
        <ListViewItem Content="Third Item"/>
        <ListViewItem Content="Forth Item"/>
    </ListView>
</Grid>
```

在你的`MainPage`中创建这样一段代码并运行，你就可以看到一个列表。

![2312ba26-029c-4f06-831d-a3e9d54b875f.png](https://storage.live.com/items/51816931BAB0F7A8!14047?authkey=AO7QXpgYo7-5DUU)

但是这样做丝毫没有体现出ListView的便捷性，因为这每一个列表项都是我们手写的。接下来，我们就要利用我们上一节课学习到的知识，使用绑定来快速创造列表视图。

## 与集合进行绑定

我们可以先创建一个List<int>的集合：

```csharp
public List<int> NumberList = new List<int>();
public MainPage()
{
    this.InitializeComponent();
    for (int i = 0; i < 1000; i++)
    {
        NumberList.Add(i);
    }
}
```

在XAML中，我们可以这样使用绑定：

```xml
<Grid>
    <ListView ItemsSource="{x:Bind NumberList}"/>
</Grid>
```

看，是不是很简单？现在运行一下应用：

![e5889a2e-f8a4-4d33-a61b-95e31acf6535.png](https://storage.live.com/items/51816931BAB0F7A8!14048?authkey=AO7QXpgYo7-5DUU)

现在你就拥有了一个包含1000个列表项的列表视图了，想想如果这是你手写的话……简直是噩梦。

到了现在，你已经学会了`ListView`的基本用法，**创建视图**和**绑定数据源**。这是我们的基础，接下来我们要学一些更深入的东西了。

:::tip
**情景模拟**：

我们现在正在创建一个聊天列表，对于聊天列表而言，列表项的数目不是固定的，时不时就会有一个新的聊天加入进聊天列表，或者某个聊天会话被删除。
:::

对于这个情景，现在你知道可以使用ListView，并在Code-Behind中创建一个数据源绑到ListView上，这样ListView可以帮助我们批量创建列表项。那么你对数据源进行条目的增删，会反映在UI上吗？

让我们来做个实验：

使用我们之前创建的代码，只不过列表项的数目要少一点，我们生成5个。然后在Page中加两个按钮，一个是**添加条目**，一个是**删除条目**。

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition />
        <RowDefinition Height="Auto"/>
    </Grid.RowDefinitions>
    <ListView ItemsSource="{x:Bind NumberList}"/>
    <StackPanel Orientation="Horizontal">
        <Button Content="添加条目" x:Name="AddButton" Click="AddButton_Click"/>
        <Button Margin="15,0,0,0" Content="删除条目" x:Name="DeleteButton" Click="DeleteButton_Click"/>
    </StackPanel>
</Grid>
```

```csharp
public List<int> NumberList = new List<int>();
public MainPage()
{
    this.InitializeComponent();
    for (int i = 0; i < 5; i++)
    {
        NumberList.Add(i);
    }
}

private void AddButton_Click(object sender, RoutedEventArgs e)
{
    NumberList.Add(NumberList.Count);
}

private void DeleteButton_Click(object sender, RoutedEventArgs e)
{
    NumberList.Remove(NumberList.LastOrDefault());
}
```

- **添加按钮**：向数据源`NumberList`中追加一个整数
- **删除按钮**：移除数据源`NumberList`中的最后一个数

现在我们可以运行软件试一试：

:::warning
点击添加或删除，UI都没有反应，但是在debug中可以发现，数据源确实被修改了。
:::

这是为什么呢？

看上去和我们上节课举的一个示例很像啊，这应该是数据源的变化没有对UI进行通知，导致UI没有改变的问题。

那么我们要怎么解决呢？

使用`INotifyPropertyChanged`接口？

答案是否定的。

为什么？

## 使用 ObservableCollection

注意这个接口的名字，`PropertyChanged`，重点在**Property**，我们的`NumberList`是Property吗？

目前它是MainPage的一个变量，也许你可以按照Property的方式写，但对于列表而言，它的变化有两个层级：

1. **列表项数目的变化**，比如我添加了一个新的列表项
2. **列表项的变化**，比如我修改了某一个列表项的值

列表项的变化可以使用`INotifyPropertyChanged`来解决（这是后面的内容），而列表项数目的变化则不然，我们需要一个新的接口`INotifyCollectionChanged`。这个接口就专门用来处理这个列表项数目变化这件事。

在WPF中，微软提供了一个名为`ObservableCollection<T>`的集合类型，UWP也继承过来了。你可以这样理解：

**这个`ObservableCollection<T>`就是一个继承了`INotifyCollectionChanged`接口的`List<T>`**

现在要做什么，我们就应该清楚了。

使用`ObservableCollection`来替代我们最初的`List`。

```csharp
public ObservableCollection<int> NumberList = new ObservableCollection<int>();
public MainPage()
{
    this.InitializeComponent();
    for (int i = 0; i < 5; i++)
    {
        NumberList.Add(i);
    }
}
```

再次运行应用，点击添加按钮及删除按钮，一切工作正常。

## 小结

尽管在文章开头我放了一张不错的聊天列表设计图片，但这节课我们并没有牵扯到关于设计的东西，因为这会是我们下节课的主要内容。这节课主要告诉你，怎么创建一个列表视图，以及怎么将数据源绑定到列表视图中。

在结尾，我提到了关于可观察集合`ObservableCollection`的用法，通常来说，对于那些列表项可变的场景，我们都会使用`ObservableCollection`而不是用`List`或者其它集合（比如`Array`, `Dictionary`），因为`ObservableCollection`继承了`INotifyCollectionChanged`接口，省去了我们自己写的工夫。

另外，本章的内容可以直接套在GridView上，尽管它们在UI表现上有所不同，但是实质是相通的。

## 延伸阅读

[列表视图和网格视图](https://docs.microsoft.com/zh-cn/windows/uwp/design/controls-and-patterns/listview-and-gridview)



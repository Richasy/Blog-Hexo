---
title: UWP小书 - ListView 和 GridView (中)
lang: zh
type: post
date: 2020/03/27 10:00:00
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- DataTemplate
- ItemContainerStyle
:::

在上一章中，我们简单地介绍了一下ListView/GridView的好处，以及如何使用集合进行绑定。

但是在实际项目中，我们很难只绑定一个字符串集合，通常都是绑定类集合，比如`ObservableCollection<User>`等。我们的需求有时候也不会仅显示字符串这么简单，可能会显示一张张卡片，或者像QQ那样显示一个聊天列表，那么这该如何完成呢？

<!--more-->

## DataTemplate

我们知道，ListView/GridView可以根据集合批量创建视图，但这个视图具体长啥样，全靠`DataTemplate`决定。

顾名思义，`DataTemplate`就是一个模板，`DataTemplate`和`ListView`的关系，就像墙面和瓷砖的关系一样。`ListView`提供一个容器，`DataTemplate`决定每一个条目的具体模样。

在上一章中，我们绑定了一个字符串集合，但没有提供`DateTemplate`，现在我们书接上文，继续添加一个`DataTemplate`:

```xml
<ListView ItemsSource="{x:Bind StringCollection}" HorizontalAlignment="Center">
    <ListView.ItemTemplate>
        <DataTemplate>
            <Grid BorderBrush="LightBlue" BorderThickness="0,0,0,1" Padding="0,0,0,5">
                <TextBlock Text="{Binding}" FontWeight="Bold" FontSize="25" FontStyle="Italic"/>
            </Grid>
        </DataTemplate>
    </ListView.ItemTemplate>
</ListView>
```

这里需要解释两个地方：

1. `DataTemplate` 算是一种资源，一种类型，并不是说ListView有一个名为`DataTemplate`的属性，只能说ListView有一个类型为`DataTemplate`的名为`ItemTemplate`的属性。这也是DataTemplate写在ListView.ItemTemplate中的原因。
2. 在绑定字符串的时候，我用了这样的语句`Text="{Binding}"`，这种写法表示绑定DataTemplate代表的数据上下文（DataContext）。

:::tip
你可能不太理解什么是数据上下文，当然也怪这个翻译的含义有些不太明确。简单来说，我们知道属性、方法，而笼阔这些属性、方法的就是数据上下文（不单单指某个类）。在上面的实例中，对于`DataTemplate`而言，所谓的数据上下文，指的就是单个的`String`。我们是依托一个字符串集合创建的列表视图，每一个字符串都对应一个视图，那么我们就可以说，视图对应的数据上下文就是那个视图对应的字符串。
:::

现在运行我们可以看到效果：

![01.png](https://i.loli.net/2020/03/27/aILCXFYxu1iJcOG.png)

ListView忠实渲染了我们预设的模板，并填充了数据。这跟我们上一章中不添加`DataTemplate`所显示的默认效果相比，是不是要好看一些呢？

但是我提出了一个新的需求：

我们注意到，对于单个列表项而言，字符串的位置是偏左的，如果我要让它居于列表项的中间位置呢？

你会发现，在DataTemplate中，你不管是修改`Grid.HorizontalAlignment`还是`TextBlock.HorizontalAlignment`都没有用，这又是为什么呢？
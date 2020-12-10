---
title: UWP经验 - 防止应用内存泄漏的两种方法
lang: zh
type: post
date: 2020/12/10 18:20:00
categories:
- UWP
tags:
- uwp
- memory
- 内存
- 页面实例
---

有时候我们在开发时会遇到一种奇怪的现象。随着我们重复地导航页面，内存占用持续上涨，永无止境。在使用一些UWP应用时你也可能发现，伴随着你操作次数的增多，切换页面的频次增加，任务管理器中的内存占用将会膨胀到一个可怕的地步。

一个简单的40M的应用启动内存，可能会增加至800M这个地步（不是Chrome，对它来说这是正常现象）

<!--More-->

简单来说，UWP是基于页面组织导航结构的，而出现内存泄漏的原因，除开代码自身的问题，绝大部分是由于页面没有及时释放导致的。

UWP中的页面并没有一个Dispose的释放方法。在你导航离开某个页面时，你会注意到这个页面的确触发了 `Unloaded` 事件，但这个事件仅表明当前页面从 `Visual Tree` 上移除，它依然可能留在内存里，最骚的是垃圾回收（GC）还无法回收。

**出现这种情况的核心就在于 `事件 (Event)` 。**

## 简单的测试

我们可以来做个demo进行测试：

> 我们会创建两个页面 `Page1` 和 `Page2`，通过重复的导航来创建多个页面的实例，通过手动调用GC来查看垃圾回收是否能正确回收不在当前 Frame 下的页面实例。

**实例计数器**

```csharp
public class InstanceCounter : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    private readonly CoreDispatcher dispatcher;
    private int page1InstanceCount = 0;
    private int page2InstanceCount = 0;

    public int Page1InstanceCount { get => page1InstanceCount; }
    public int Page2InstanceCount { get => page2InstanceCount; }

    public InstanceCounter(CoreDispatcher dispatcher)
    {
        this.dispatcher = dispatcher;
    }

    public void IncrementPage1()
    {
        Interlocked.Increment(ref this.page1InstanceCount);
        this.RaisePropertyChanged(nameof(Page1InstanceCount));
    }

    public void IncrementPage2()
    {
        Interlocked.Increment(ref page2InstanceCount);
        this.RaisePropertyChanged(nameof(Page2InstanceCount));
    }

    public void DecrementPage1()
    {
        Interlocked.Decrement(ref page1InstanceCount);
        this.RaisePropertyChanged(nameof(Page1InstanceCount));
    }

    public void DecrementPage2()
    {
        Interlocked.Decrement(ref page2InstanceCount);
        this.RaisePropertyChanged(nameof(Page2InstanceCount));
    }

    private void RaisePropertyChanged(string propertyName)
    {
        IAsyncAction ignored = this.dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
        {
            this.PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        });
    }
}
```

**初始化计数器**

```csharp
//App.xaml.cs

private InstanceCounter instanceCounter;

public static InstanceCounter GetCurrentInstanceCounter()
{
    return ((App)App.Current).instanceCounter;
}

protected override void OnLaunched(LaunchActivatedEventArgs e)
{
    //...
    if (rootFrame == null)
    {
        rootFrame = new Frame();
        // ...
        this.instanceCounter = new InstanceCounter(rootFrame.Dispatcher);
    }
    //...
}
```

**页面1**

XAML

```xml
<Grid>
    <TextBlock>Page 1</TextBlock>
</Grid>
```

XAML.cs

```csharp
public Page1()
{
    this.InitializeComponent();
    App.GetCurrentInstanceCounter().IncrementPage1();
}

~Page1()
{
    App.GetCurrentInstanceCounter().DecrementPage1();
}
```

**页面2**

XAML

```xml
<Grid>
    <TextBlock>Page 2 (this page leaks memory)</TextBlock>
</Grid>
```

XAML.cs

```csharp
UISettings settings;
public Page2()
{
    this.InitializeComponent();
    App.GetCurrentInstanceCounter().IncrementPage2();

    settings = new UISettings();
    
    settings.ColorValuesChanged += this.OnColorValuesChanged; 
}

~Page2()
{
    App.GetCurrentInstanceCounter().DecrementPage2();
}

private void OnColorValuesChanged(UISettings sender, object args)
{
}
```

**主页 (MainPage)**

XAML

```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition />
    </Grid.RowDefinitions>
    <StackPanel Orientation="Horizontal">
        <Button Margin="0,0,10,0" Click="Page1_Click">Go to Page 1</Button>
        <Button Margin="0,0,10,0" Click="Page2_Click">Go to Page 2</Button>
        <TextBlock Margin="0,0,10,0"><Run>Page 1:</Run><Run Text="{x:Bind instanceCounter.Page1InstanceCount, Mode=OneWay}" /><LineBreak /><Run>Page 2:</Run><Run Text="{x:Bind instanceCounter.Page2InstanceCount, Mode=OneWay}" /></TextBlock>
        <Button Margin="0,0,10,0" Click="GC_Click">GC Collect</Button>
    </StackPanel>
    <Frame x:Name="InnerFrame" Grid.Row="1" />
</Grid>
```

XAML.cs

```csharp
private InstanceCounter instanceCounter = App.GetCurrentInstanceCounter();

public MainPage()
{
    this.InitializeComponent();
    this.InnerFrame.Navigate(typeof(Page1));
}

private void Page1_Click(object sender, RoutedEventArgs e)
{
    this.InnerFrame.Navigate(typeof(Page1));
}

private void Page2_Click(object sender, RoutedEventArgs e)
{
    this.InnerFrame.Navigate(typeof(Page2));
}

private void GC_Click(object sender, RoutedEventArgs e)
{
    GC.Collect();
}
```

## 测试结果

我们可以通过点击按钮，重复在 `Page1` 和 `Page2` 间导航，你能观察到计数器可以正确记录你创建页面实例的个数。当我们点击 `GC Collect` 按钮时。`Page1` 的计数可以被清零（或者降为1），但 `Page2` 的计数没有下降。这说明 `Page2` 的实例并没有被销毁。

## 解释

原因就在于我们在 `Page2.xaml.cs` 中对 `UISettings` 附加的事件回调并没有被移除，这导致引用持续存在，无法被垃圾回收。

所以当我们添加移除事件的代码后，垃圾回收就可以正常工作了。

*（尝试将下面这段代码写在 `Page2.xaml.cs` 中）*

```csharp
protected override void OnNavigatedFrom(NavigationEventArgs e)
{
    settings.ColorValuesChanged -= this.OnColorValuesChanged;
    base.OnNavigatedFrom(e);
}
```

有趣的是，当我们在 XAML 中为我们的控件附加事件的时候却没有出现这种情况，比如我在页面中添加了一个 Button ，附加了一个 Click 事件回调，并没有多余的操作，但当我离开页面后，这个页面实例却可以正常回收（你可以做个 demo 来验证）。

我尝试寻找可能的原因。

我们知道，在我们编写XAML代码的同时，Visual Studio 会帮助我们生成一些中间代码，这些代码存放在项目根目录下的 `obj/[Architecture]/Debug` 文件夹中。

在该文件夹内，找到我们Page对应的代码文件，比如 `Page2.g.cs` 和 `Page2.g.i.cs`，我们能接触到UWP项目更深层次的东西。在 `Page2.g.i.cs` 中，我们能发现一个方法：

```csharp
partial void UnloadObject(global::Windows.UI.Xaml.DependencyObject unloadableObject);
```

它应该与我们解绑事件有关，但到此止步，我无法在项目本身获取更多的消息了。我们可以将其理解为通过XAML附加的事件，UWP会根据控件对应的 `connectionId` (在 `Page2.g.cs`中有，在同级目录的`Page2.xaml`中定义) 查找对应的控件并移除事件。

## DataTemplate 中的暗坑

`DataTemplate` 是 UWP 应用开发中绕不开的一个重要部分，我们使用 `ListView`, `GridView` 等控件都会用到它。那你是否尝试过这种操作呢：

> 在 `Page.Resources` 中定义了一个 `DataTemplate` ，在这个数据模板中有一个 `Button` , 你为其添加了一个 `Click` 事件回调，该回调写在了所属的 Page 之中

如果是，那么恭喜你，你这个页面将难以被回收了。

前文我们提及，在 XAML 中创建的事件回调会在控件或页面 `Unloaded` 时移除，但到了 `DataTemplate` 这里就变成了一个特例。

其核心就在于，`DataTemplate` 只是个 Template ，它并不是真正定义在页面中的控件。

想一想，当你给 DataTemplate 中的控件命名时，你能在当前页的 Code-behind 中直接拿到它吗？

这其实就说明了这样一个问题，当页面 Unloaded 的时候，我们无法通过简单的ID查询找到对应的按钮了。因为你定义在DataTemplate里的按钮此时可能已经“复制”了好几十个，抑或是由于数据源缺失一个都没有。

此时我们附加了事件，最终却无法移除，这就回到了我们最初的情况，由于引用一直存在，导致页面无法被释放。

## 我们该怎么做？

1. 时刻注意移除事件

我们最好将附加事件回调的操作放在 `Loaded` 或 `OnNavigatedTo` 事件之中，在与之对应的事件（`Unloaded` 或 `OnNavigatedFrom`）中移除事件。

同时为了避免意外，在添加事件时先尝试移除事件，以避免重复添加。

```csharp
// do something...
settings.ColorValuesChanged -= this.OnColorValuesChanged;
settings.ColorValuesChanged += this.OnColorValuesChanged;
```

2. 将需要事件回调的 `DataTemplate` 包装成 `UserControl`

简单来说就是将UI界面和事件回调打包，对外暴露一个依赖属性用来接收数据，其它的逻辑都在内部完成。这样事件的自动移除就会有迹可循。

3. 谨慎使用外部控件

最好使用开源的控件。当一个控件内部存在没有移除的事件时，是会牵连到使用它的页面的，这同样会导致页面无法释放，而且定位起来会很麻烦。选择开源控件也只是能帮助你判断问题是否出现在这个引用的控件上。

4. 尝试缓存页面

如果内存占用的飙升来源于不断创建新的页面实例而旧的实例无法释放。那么我们就保留一个页面实例不再创建新的实例不就行了吗？

这时候我们就可以缓存页面：

```csharp
public SomePage()
{
    this.InitializeComponent();
    NavigationCacheMode = NavigationCacheMode.Enabled;
}
```

相比起时刻小心留意，缓存页面是一个一劳永逸的解决方案。但缺点也摆在那里。缓存页面实例意味着你放弃了主动释放页面实例，应用的内存占用会有一个上限（在你缓存了所有页面之后），但是却很难降下来。

但对于一些不方便将 `DataTemplate` 包装成 `UserControl` 的场景下，缓存页面是一个相对不错的解决方案。

:::tip
这里说的场景有很多种，举例而言，如果在你的DataTemplate的事件回调中需要访问当前页面的一些资源（比如控件，或者定义的变量）。如果定义成独立的 `UserControl` ，这一块数据就拿不到了。

在这种情况下，缓存页面问题不大。
:::
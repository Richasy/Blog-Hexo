---
title: UWP小书 - 绑定初步
lang: zh
type: post
date: 2019/10/18 22:00:00
categories:
- UWP
tags:
- uwp
---

:::tip
本章涉及知识点：
- 创建绑定的流程
- UI通知
:::

绑定是UWP应用的核心知识，熟练运用绑定，可以极大地减轻我们的工作量，同时让我们的代码的可维护性更优秀，即所谓`健壮的代码`。

<!--more-->

## 建立一个简单的绑定

现在我们提出一个情景，假设我们有一个`Person`类，里面包含一些人名、年龄之类的字段，现在我们要在界面上用表格展示这个类的内容，就像图中所示：

![2d87cc3c-7221-4351-87f9-4c4b6e696512.png](https://storage.live.com/items/51816931BAB0F7A8!13877?authkey=AO7QXpgYo7-5DUU)

现在让我们来构建这个类，这个类包含了三个基础属性 `Name`, `Age`, `Sex`：

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string Sex { get; set; }
    
    public Person() { }
    public Person(string _name,int _age,string _sex)
    {
        Name = _name;
        Age = _age;
        Sex = _sex;
    }
}
```

既然创建了类，我们便需要一个实例，打开我们的`MainPage.xaml.cs`，在`MainPage`的构造函数上方新建一个`_myPerson`。

```csharp
public Person _myPerson = new Person("Richard", 24, "男");
```

实例创建完成，它就是我们的作为我们绑定依据的数据源了。

现在创建我们的布局，打开`MainPage.xaml`，输入以下代码：

```xml
<Page.Resources>
    <Style TargetType="TextBlock" x:Key="TipBlockStyle">
        <Setter Property="FontSize" Value="20"/>
        <Setter Property="Foreground" Value="LightGray"/>
    </Style>
    <Style TargetType="TextBlock" x:Key="ContentBlockStyle">
        <Setter Property="FontSize" Value="20"/>
        <Setter Property="FontWeight" Value="Bold"/>
        <Setter Property="Foreground" Value="White"/>
    </Style>
</Page.Resources>

<Grid>
    <Grid BorderBrush="Gray" HorizontalAlignment="Center" VerticalAlignment="Center">
        <Grid.RowDefinitions>
            <RowDefinition Height="50"/>
            <RowDefinition Height="50"/>
            <RowDefinition Height="50"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="70"/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        
        <TextBlock Text="名字" Style="{StaticResource TipBlockStyle}"/>
        <TextBlock x:Name="NameBlock" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>

        <TextBlock Text="年龄" Grid.Row="1" Style="{StaticResource TipBlockStyle}"/>
        <TextBlock x:Name="AgeBlock" Grid.Row="1" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>

        <TextBlock Text="性别" Grid.Row="2" Style="{StaticResource TipBlockStyle}"/>
        <TextBlock x:Name="SexBlock" Grid.Row="2" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>
    </Grid>
</Grid>
```

这样就创建好了布局，这里并没有涉及其它的知识，如果你看过我前面的教程，相信这些对你是信手拈来，再不济也能理解。

可问题来了，我们布局创建好了，可关键的数据没写上啊。`NameBlock`, `AgeBlock`和`SexBlock`都是空的。

这时候就要建立绑定了，将我们`_myPerson`中的数据填充到这些**TextBlock**里面。

绑定有两种，一种是`x:Bind`，一种是`Binding`，他俩各有千秋，我们之后再讲，这里就用`x:Bind`。

既然我们要让数据显示出来，那么我们就要把Person类里的对应属性绑定到`TextBlock.Text`属性上，且看语法：

```xml
<TextBlock Text="{x:Bind _myPerson.Name}" />
```

并不难理解，花括号表示表达式，`x:Bind`相当于关键字，`_myPerson.Name`就是需要绑定的数据。

将那三个`TextBlock`统统绑定上，就像这样：

```xml
<TextBlock x:Name="NameBlock" Text="{x:Bind _myPerson.Name}" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>

<TextBlock x:Name="AgeBlock" Text="{x:Bind _myPerson.Age}" Grid.Row="1" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>

<TextBlock x:Name="SexBlock" Text="{x:Bind _myPerson.Sex}" Grid.Row="2" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>
```

现在运行应用，你就可以看到最开始图片中的效果了。

:::tip
**绑定的好处**  
  
说起绑定的好处那还真是不少，最大的莫过于减少代码量。

回想一下我们刚才创建绑定的步骤，再来想想如果没有绑定我们要怎么做。

首先，你有一个动态的数据源，也即其属性是会变化的，所以你就不能在XAML中写死。你需要给控件命名，然后在代码中使用`NameBlock.Text="xxx"`来修改控件属性。

每变化一次你就要写一次。假设我们有10个`NameBlock`，那就要写10次，这实在是很累的一件事情。

但有了绑定，情况就不同了，尽管我们还没涉及到修改数据那一块，但是通过绑定，我们只需要修改数据源，那么与数据源绑定的所有控件属性都会被修改，可以说是省事儿了不少，免得自己一一赋值了。
:::

## 修改数据

从上面的步骤来看，我们只完成了赋值工作。通常来说，既然是绑定，那么只要我们修改了数据源，UI就会同步变化，但事实真是如此吗？

我们可以添加一个按钮来验证一下：

```xml
<Button HorizontalAlignment="Center" Margin="0,20,0,0" Content="点我修改数据" Click="Button_Click"/>
```

```csharp
private void Button_Click(object sender, RoutedEventArgs e)
{
    _myPerson.Name = "云之幻";
}
```

运行之后，我们会发现`NameBlock`的值并没有任何变化。这又是为什么呢？

这里就要谈到两个问题了：

**1. UI如何知道数据源修改了**

我们以为绑定就是把控件属性和数据源牢牢绑在了一起，只要数据源改了，UI就会跟着改。这么理解不能算错，但需要我们做一些额外的工作。即在数据修改时通知UI界面我这个数据改了，你也要更新了。

是的，这个通信需要我们自己来写。

现在我们修改`Person`类：

```csharp
public class Person:INotifyPropertyChanged
{
    private string _name;

    public string Name
    {
        get => _name;
        set
        {
            _name = value;
            OnPropertyChanged();
        }
    }
    public int Age { get; set; }
    public string Sex { get; set; }

    public Person() { }
    public Person(string _name,int _age,string _sex)
    {
        Name = _name;
        Age = _age;
        Sex = _sex;
    }

    public event PropertyChangedEventHandler PropertyChanged;
    public void OnPropertyChanged([CallerMemberName]string propertyName = "") 
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

}
```

UWP有这样一个接口`INotifyPropertyChanged`，专司UI通知，为了实现它，你至少要一个`PropertyChanged`事件，还要有一个处理该事件的方法（类中的`OnPropertyChanged`），有了这个方法之后，你就不能再对需要UI通知的属性做简单的`{get; set;}`了，而是要重写getter和setter，特别是在setter中加入属性更改通知。

但加了这个还不够，如果你再运行应用，你会发现结果并没有什么不同。

**2. 创建持久连接**

找对象有三种，一者约，二者恋，三者婚，羁绊程度逐步加深，绑定也是同样，它有三种方式：

1. `OneTime`
2. `OneWay`
3. `TwoWay`

我们用的`x:Bind`，默认就是`OneTime`，在UI开始创建的时候绑一次，取了数据源的值作为属性值之后，就不再管后续的修改了，颇有些提上裤子不认人的意思。

为了能够让控件和数据源之间建立一个稳定的通道，让数据源的修改能在UI上及时反映出来，我们需要调整绑定的方式。

请修改`NameBlock`:

```xml
<TextBlock x:Name="NameBlock" Text="{x:Bind _myPerson.Name, Mode=OneWay}" Grid.Column="1" Style="{StaticResource ContentBlockStyle}"/>
```

将`Mode`改为`OneWay`之后，再运行应用，你就可以发现，点击按钮之后，`NameBlock`的值就被改过来了。

现在简单地说说绑定方式。

`OneTime`是个怎么样的绑定行为，你已经体验过，这里不再赘述。

`OneWay`表示数据单向绑定，这个单向，指的是从数据源流向UI。假设你有一个`TextBox`，你将Text属性绑定好了，那么在你修改数据源的时候，TextBox里面的Text会改变。但当你直接编辑TextBox里的内容时，数据源的数据并不会修改。

如果你想要修改，那就要使用`TwoWay`。

`TwoWay`，就是双向绑定。换句话说，改数据源，UI会变；改UI，数据源会变。

不同的绑定方式有不同的用处，这个在我们后面上手做应用的时候会带着大家来熟悉。

## 小结

本章带着大家初步上手了绑定，在做UWP应用的过程中，绑定可以说是贯穿始终，是非常重要的一个内容。

凭着本章学到的内容，你可以让前后端更加分离，更好解耦，更有助于你的代码维护。

但是想把绑定玩出花活，那还需要后面的进阶知识。

## 延伸阅读

- [Data Binding](https://docs.microsoft.com/zh-cn/windows/uwp/data-binding/data-binding-quickstart)


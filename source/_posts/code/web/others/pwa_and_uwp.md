---
type: post
lang: zh
title: PWA 与 UWP 的结合
date: 2019/7/1 9:29
categories:
- 网页
- PWA
tags:
- pwa
- uwp
- web
---

:::tip
早前，微软为了挽回UWP的颓势，曾推进PWA上架微软应用商店的工作。配合微软进行转制的有Uber、Twitter等知名大厂的网页应用。这些经过转制的应用看起来和网页端并无二致，但却可以调用`NativeAPI`，在某些地方达到甚至超越了nodejs的效果，而这种转制，也让PWA在Windows下更为强大。

本章便是来探讨PWA是如何与UWP相结合，产生 **1+1>2** 的效果的。
:::

<!--more-->

## 什么是 PWA

PWA，全称 `Progressive Web App`，翻译过来就是渐进式网页应用。所以PWA首先要是一个网页，在此基础上，如果浏览器或者其他的渲染器支持，才会变成一个应用。这也正是其**渐进式**的由来。

从组成结构上来说，PWA是对原始网页的一个补充。这种补充不是破坏式的，因为只需要添加两个文件，这两个文件分别是`Service Worker`和`Manifest`。甚至极端一些，如果你只对PWA所带来的强大后台功能感兴趣，那么`Manifest`都可以不要。

通常来说，PWA更喜欢移动端。它是App-Like，像本地应用却又不是本地应用。比本地应用更轻，卸载也更为方便。

但是在桌面端，PWA便不甚突出。相比起应用，在浏览器中打开几个标签页显然更为方便，如果没有一些特殊的功能，那么是否集成PWA则显得不那么重要。

显然，UWP下WinRT的加入，便是为PWA在桌面端赋予了新的能力。尽管在新的VS2019中已经不再支持。

## 以 PWA 为基础的 UWP 应用

UWP是一个平台，而最原汁原味的应用开发自然是采用`XAML`+`C#`，除此之外，Javascript搭配HTML也是一个不错的实现。但不管开发采取何种方式，在与本机API交互时，都需要依托WinRT API，相当于一个中间层。

当PWA应用运行于UWP平台之上时，启动它的进程并非浏览器进程，而是名为`wwahost.exe`的进程。

Javascript能调用绝大部分WinRT API，但有一些平台独有的，比如`Windows.UI.XAML`命名空间下的API就不能调用。这意味着PWA应用并不能调取Acrylic Brush来实现Fluent Design中的动效及光效，不过前端UI库极多，达到类似的效果也并不困难。

UWP对PWA的赋能，更多地体现在平台特色功能上。随便举几个例子：

- 本地文件IO
- Microsoft服务的访问
- 机器学习
- 设备调用（比如照相机、陀螺仪等）
- 通知推送
- Cortana

上述功能只是诸多特色功能中的一部分，这得益于Windows10带来的强大功能，以及WinRT作为中间层提供的良好兼容能力。

## 从现有 PWA 项目中建立 UWP 应用

假设你已经写好了一个PWA应用，包括了`Service Worker`，Manifest有没有并不重要。

那么如何根据这个项目创建一个UWP应用呢？非常简单，甚至不需要你修改现有代码。

打开**VS2017** *（必须要是这个版本的Visual Studio）*。新建一个`Progressive Web App`项目。  
![531a0d1c-ae65-41ca-b240-08d33b61a39f.png](https://storage.live.com/items/51816931BAB0F7A8!13425?authkey=AO7QXpgYo7-5DUU)

我们可以观察新项目的目录结构，可以发现，新项目仅有几个标注为error错误页面，和一些UWP项目自带的图标文件、应用清单文件等。

![7a4018c0-f653-4779-9d4d-d29749254705.png](https://storage.live.com/items/51816931BAB0F7A8!13430?authkey=AO7QXpgYo7-5DUU)

这和我们平常的开发可不一样，看样子VS并没有为我们提供`index.html`等配套文件。奇怪，这些文件难道要我们自己创建吗？

当然不是。

甚至你根本不需要添加任何文件。

那如何让它工作呢？

答案就在应用清单中。

打开`package.appxmanifest`，秘密就在Application -> Start page 选项中。

如果你的页面上线了，直接在这里输入你的网页网址，点击运行，它就可以工作了。

![b1850053-6613-40c8-bdfc-a102b4050bf5.png](https://storage.live.com/items/51816931BAB0F7A8!13426?authkey=AO7QXpgYo7-5DUU)

想新建一个以PWA为基础的UWP应用，可以说是非常简单了。

如果你有UWP的基础，相信对你来说，配置应用图标，改写应用名这些操作应该是驾轻就熟，无需赘述了。

## 代码中集成WinRT API

对于从现有项目中新建UWP应用而言，并没有什么值得称道的地方。因为这看起来就是一个浏览器套壳应用罢了。

但如果你打算将PWA应用转制为UWP，那么你感兴趣的地方就绝不仅于此。

尽管UWP应用本身反响平平，但是在功能性上仍比PWA应用要强上很多，比如文件IO。

### 调用本地文件操作API的例子

> 对于一个网页应用，`能读不能写`几乎是常态。你可以通过Input标签来获取本机文件，但很显然，如果你想保存什么文件到计算机上，就要麻烦很多了，甚至要走许多弯路。但是依托于UWP，我们可以直接调用本地API帮助我们自如地实现文件的读写。

新建一个简单的项目（你可以使用喜欢的编辑器，比如VS Code），项目结构如下所示：

```
IO
|- index.html
|- sw.js
```

在`index.html`中，我们新建两个按钮及一个文本框，用作我们的UI元素。

```html
<div class="container">
    <div class="left">
        <button id="import">导入</button>
        <button id="export" style="margin-top: 20px">导出</button>
    </div>
    <div class="right">
        <textarea id="contentBox" placeholder="请输入内容"></textarea>
    </div>
</div>
```

接下来注册我们的`serviceWorker`。

```javascript
if ("serviceWorker" in navigator) {
   navigator.serviceWorker.register("/sw.js").then(function (registration) {
     console.log("Service Worker registered with scope:", registration.scope);
   }).catch(function (err) {
     console.log("Service worker registration failed:", err);
   });
}
```

这些算是基本的准备工作，为求简便，脚本可以直接写在`<body>`标签中，尽管这并不符合书写规范。

接下来我们为两个按钮绑定事件，将下面的代码写入到html文件中：

```javascript
let win = window.Windows;
let importButton = document.querySelector('#import');
let exportButton = document.querySelector('#export');
// 导入文件，调用了FileOpenPicker
importButton.addEventListener('click', function () {
    if (win) {
        let sto = win.Storage;
        let picker = new sto.Pickers.FileOpenPicker();
        let input = document.querySelector('#contentBox')
        picker.SuggestStartLocation = sto.Pickers.PickerLocationId.documentsLibrary;
        picker.fileTypeFilter.push('.txt');
        picker.pickSingleFileAsync()
        .then((file) => {
            sto.FileIO.readTextAsync(file).then((str) => {
                input.value=str;
            })
        })
        .catch((e) => {
            console.log('读取文件失败')
        })
    }
})

// 导出文本至文件中，调用了FileSavePicker
exportButton.addEventListener('click', function () {
    if (win) {
        let sto = win.Storage;
        let text = document.querySelector('#contentBox').value;
        if (text) {
            let picker = new sto.Pickers.FileSavePicker();
            picker.defaultFileExtension = '.txt';
            picker.suggestedFileName = 'Test.txt';
            picker.fileTypeChoices.insert('文本文件', ['.txt']);
            picker.suggestStartLocation = sto.Pickers.PickerLocationId.documentsLibrary;
            picker.pickSaveFileAsync()
            .then((file) => {
                sto.FileIO.writeTextAsync(file,text);
            })
            .catch((e) => {
                console.log('保存文件失败')
            })
        }
    }
})
```

如果你之前开发过传统UWP应用，那么这里面的一些类你应该很熟悉，比如FileOpenPicker-文件选择器、FileSavePicker-文件保存器、FileIO-文件静态操作类等。

> 相比起C#，Javascript的调用显然要复杂一些，需要写出完整的类名，即包含了命名空间的类名。

所有这些WinRT API都被放在了window.Windows之中，所以每次调用的时候，都是先从这里开始。

但是有一个问题，原生JS中并没有window.Windows这个对象，我们也从来没有创建过它，那为什么通过它可以调用本机API呢？

答案还是在我们的PWA-UWP项目上。

### 为PWA项目开启WinRT支持

在进入到我们最开始用VS创建的PWA项目之前，你需要先运行一个本地服务器，让我们刚刚写的文件IO的例子可以运行。

我使用的是本机的IIS服务器，前端也有很多其他的选择，比如`http-server`。喜欢使用哪个就使用哪个，这无关紧要。

起一个本地服务器的目的也仅在于能通过http链接访问到我们的 `index.html` 文件。

服务器跑起来之后，将对应的链接写在 `package.appxmanifest` 的`Start page`中。

这是我们的准备工作。

如果你这个时候直接运行项目，你会发现点击按钮一点反应也没有。这很正常，因为你并没有开启该UWP项目的特殊权限。

想要让window.Windows工作，需要在`package.appxmanifest`下的`Content URIs`中添加网站所对应的链接，并将 `WinRT Access` 设置为`All`。

![4e046625-a9f0-4f37-a655-ab013cd91641.png](https://storage.live.com/items/51816931BAB0F7A8!13427?authkey=AO7QXpgYo7-5DUU)

这个时候重新运行项目，你就会发现当你点击按钮的时候，文件选择器就会顺利弹出了。

---

### 小结

我们完成了一次简单的实验。即通过Javascript调用本机API来帮助我们实现文件的读写。

尽管在实际编写中需要频繁查文档而显得略有些繁琐，但总体的结果是好的。Javascript和WinRT API的交互也很顺畅。

但仍有一些需要注意的问题。

**首要的便在于命名。**

C#中的命名遵循帕斯卡规范，即每个单词首字母都大写，而Javascript多采用驼峰命名。在调用WinRT API时，微软更改了方法和部分属性的命名方式，从帕斯卡变为了小驼峰。在没有代码提示的情况下，极易写错，这是一个大坑，在书写的时候应遵循以下规律：

1. 命名空间和类是帕斯卡写法
2. 类的成员（包括方法和属性）是驼峰命名
3. 事件名称纯小写

**其次是支持的功能有限。**

尽管大多数特色功能都得到了支持，但只能作为补充。有些内容，比如`Windows.UI.Controls`里的内容就不可用。关于可用及不可用的API，你可以参照[WinRT文档](https://docs.microsoft.com/zh-cn/microsoft-edge/windows-runtime)。

**最后则是工作范围问题。**

你也看到了，我们调用WinRT API的途径是`window.Windows`，而在Service Worker中，`window`变量是不存在的，有的只是`self`，而`self.Windows`并不存在。所以这意味着一件事，在Service Worker中无法使用WinRT API。

所以，我们并没有可能将Service Worker变成一个Runtime Component。

## 与Service Worker结合推送消息

尽管我们说，试图将Service Worker变为Runtime Component是一种徒劳。但这并不意味着我们不能借助Service Worker来配合工作了。

假使我们需要用到消息推送。

在前端中，消息推送早已有之，比如`web-push`。但这种消息推送只能用作信息展示。Win10的Toast Notification能做的事情远不止于此，它还可以添加按钮以进行交互。

想要使用原生的Toast Notification，则势必要在前台进行触发。*这里的前台指的是与Service Worker相对应的主UI线程。*

按照一般PWA的开发趋势，数据处理（不涉及前台UI交互）放在Service Worker中，我们可以简化成如下处理逻辑：

**前台发起信息处理请求 --> Service Worker进行处理 --> 将处理结果发回前台 --> 前台弹出通知**

### 调用通知模块

我们要调用本地的Toast Notification，这涉及到WinRT API。

那就先把这个弹出通知的方法写出来：

```javascript
/*
 * data结构：
 * {
 *   title: string,
 *   content: string
 * }
 */
function sendMessage(data) {
    let title = data.title;
    let content = data.content;
    if (window.Windows && window.Windows.UI.Notifications) {
        let notify = window.Windows.UI.Notifications;
        let xml =
        `
        <toast launch="app-defined-string">
          <visual>
              <binding template="ToastGeneric">
                  <text hint-maxLines="1">${title}</text>
                  <text>${content}</text>
                  <image placement="appLogoOverride" hint-crop="circle" src="https://picsum.photos/48?image=883"/>
              </binding>
          </visual>
          <actions>
              <action content="查看更多细节" arguments="action=detail&amp;contentId=351" activationType="foreground"/>
          </actions>
        </toast>
        `;
        let doc=new Windows.Data.Xml.Dom.XmlDocument();
        doc.loadXml(xml);
        let notification = new notify.ToastNotification(doc);
        notify.ToastNotificationManager.createToastNotifier().show(notification);
    }
}
```

这里的通知UI我们用一个XML字符串表示，关于通知的UI结构，可以参见[官方文档](https://docs.microsoft.com/zh-cn/windows/uwp/design/shell/tiles-and-notifications/adaptive-interactive-toasts)。

我们的通知包含了**LOGO**、**标题**、**文本**和一个**按钮**。我们希望达到一个效果：

点击弹出通知的按钮，前台给予响应 *（即能捕获到对应的点击事件，并可以获取到按钮附带的参数）*

在UWP中，Toast的激活分为前台和后台两种，由于PWA-UWP项目没有后台支持，所以这里仅考虑前台的情况。

点击通知时，会激活软件，所以我们需要捕获软件的Activated事件。该事件由UWP框架提供，所以这里我们来看如何触发这一事件：

```javascript
if (window.Windows) {
  Windows.UI.WebUI.WebUIApplication.addEventListener("activated", function (activatedEventArgs) {
    if (activatedEventArgs.kind == Windows.ApplicationModel.Activation.ActivationKind.toastNotification) {
      console.log(activatedEventArgs.argument);
    }
  });
}
```

我们渲染出的网页就放在`WebUIApplication`之中，所以给这个容器注册一个`activated`事件即可。但软件激活有多种渠道，为了确保抓到我们需要的事件，需要做一个甄别，这个甄别就通过事件参数的`kind`属性来进行，对上号了，我们就处理附带的信息参数（这里只是简单的输出）

我们现在完成了需要用到WinRT API的部分，接下来的事情就交由Javascript原生API来处理了。

### 窗口与Service Worker通信

首先建立一个`sw.js`的文件，并在主页注册它：

```javascript
if ("serviceWorker" in navigator) {
   navigator.serviceWorker.register("/sw.js").then(function (registration) {
     console.log("Service Worker registered with scope:", registration.scope);
   }).catch(function (err) {
     console.log("Service worker registration failed:", err);
   });
}
```

然后我们在`<body>`中写一个按钮，用以做我们的触发器：

```html
<body>
  <button id="pushButton">推送消息</button>
</body>
```

添加按钮的事件处理方法：

```javascript
let btn = document.querySelector('#pushButton');
btn.addEventListener('click', function () {
  // 利用postMessage和serviceWorker通信
  navigator.serviceWorker.controller.postMessage('toast');
})
```

在sw.js中捕获信息，进行数据处理，并返回给前台：

```javascript
self.addEventListener('message', event => {
  if(event.data=='toast'){
    self.clients.matchAll().then(function(clients){
      clients.forEach(function(client){
          // 向前台发送信息
          client.postMessage(
            {title:'后台信息',content:'来自后台的你'}
          );
      })
    })
  }
})
```

既然serviceWorker回传了数据，那么前台就必须要接收才行：

```javascript
navigator.serviceWorker.addEventListener("message", function (event) {
    // sendMessage方法是我们最开始就定义好的
    sendMessage(event.data);
});
```

现在可以运行一下软件试试了。点击按钮，屏幕右下方就会弹出通知，点击通知上的按钮，留意控制台的输出：

![e2817abd-1f83-4110-969d-02ade71b144c.png](https://storage.live.com/items/51816931BAB0F7A8!13428?authkey=AO7QXpgYo7-5DUU)

![8435a417-65d7-44a6-87d6-94f6dce7fb4d.png](https://storage.live.com/items/51816931BAB0F7A8!13429?authkey=AO7QXpgYo7-5DUU)

## 借助Typescript实现智能提示

用Javascript写代码有一个不太好的地方，即智能提示有限。尤其是在我们写WinRT相关API的时候，我想没谁记得住那一堆堆的API，尤其是全名。

如果你不想在写的时候疯狂查文档，我想你需要试试Typescript。

Typescript也是微软家的，你可以理解为是具备了静态类型检查功能的Javascript。静态类型的语言有一个好处就是类型确定，所有的属性、方法、字段都有定义，做起代码提示的时候就方便很多。

在开始搞智能提示之前，请先安装好Typescript的相关环境，具体如何安装，并不是本文需要讲的内容，请读者自行查阅。

假设你安装了Typescript，并对其有了一定的了解，现在开始我们的操作吧：

### 新建一个WinJS的UWP项目

Typescript想实现代码提示，需要有一个定义文件，后缀名为`.d.ts`。但是在官方的`@types`库中，并不存在window.Windows相关的定义文件。

幸运的是，微软为了能让WinJS的项目可以正常工作，已经将WinRT的API定义文件写进了VS中，新建一个WinJS项目就是为了找到它。

![dc698ebd-5f66-44a7-a738-2503ba7c9e1a.png](https://storage.live.com/items/51816931BAB0F7A8!13432?authkey=AO7QXpgYo7-5DUU)

新建项目后，在项目管理列表窗口里打开所有隐藏文件，在如下目录中找到我们的目标文件：`winrtrefs.d.ts`

![8aacd944-e17b-466b-8773-434564dabab9.png](https://storage.live.com/items/51816931BAB0F7A8!13433?authkey=AO7QXpgYo7-5DUU)

### 引入定义文件

得到了`winrtrefs.d.ts`文件之后，我们将其复制到我们的PWA项目中。

:::tip
如果你已有一个项目，且不好转换到Typescript，可以另起炉灶，将你要写的逻辑代码视作模块，单独编写，单独编译
:::

新建一个ts文件，我们在顶部对`.d.ts`文件进行引入：

```typescript
///<reference path="./winrtrefs.d.ts"/>
```

### 欺骗Typescript编译器

前面两步其实很简单，就是复制+引入，做完之后你就已经拥有智能提示的能力了。

当然，这需要编辑器的支持，我个人推荐使用**VS Code**。

但是在Typescript中编写代码的时候还是会遇到问题。如果你直接书写`window.Windows`，编辑器会报错，说`Windows`并没有在`window`中定义。

这就是静态类型检查和动态语言之间的矛盾了。我们明白Windows是在UWP容器下附加到window中的变量，事先当然不好定义。

为了编译器放我们一马，这里要做一点处理：

```typescript
declare interface Window {
    Windows: any;
}
```

强行怼一个定义即可。

接下来就可以享受代码提示带来的快感了：

![15764430-b394-4e92-8182-09c7ad0677dd.png](https://storage.live.com/items/51816931BAB0F7A8!13435?authkey=AO7QXpgYo7-5DUU)

## 结语

我们举了两个例子加一个技巧，这两个例子设计到的PWA的内容少，而WinRT的内容多，因为我们这篇文章的侧重点就在于此，但真要做PWA，依然要以网页技术为主。

在传统UWP应用中，我们使用LocalSettings存放用户设置，使用本地文件来存放结构化数据。这些在网页技术中都有对应的内容，比如localStorage或indexDb。由于Service Worker中没有window，我们不能借助WinRT将PWA改造成一个完全的本地应用，事实上也完全没有必要。

WinRT API只能在前台使用，这注定WinRT API的身份只是赋能，而不是革新。

赋能何意？

借助UWP的容器，实现本地化功能，获取一些仅本地应用才能获取的权限。

但是除开这些，PWA应用本质上还是一个功能完整的网页应用。理论上不应该有任何重要功能依赖于WinRT，所以对于一个成熟的PWA项目来说，WinRT可以帮助应用获得更好的本地化体验，却不能成就它。

就如同外国领导人新春时送上的一句中文祝福，或可以博一个亲民的彩头。但说与不说，都不会对两国关系有实质性影响。

在新的Visual Studio 2019中，JS相关的UWP开发都已被移除。其实按道理来说，这种开发手段不应该这么激进地处理，考虑到微软放弃了当前的Edge浏览器，转投Chromium阵营。我猜测是原Edge团队内部出现了问题，导致Edge无法继续更新或更新频率过低（与Windows捆绑）。

而不久前，微软正在开放基于Chromium的[WebView2](https://blogs.windows.com/msedgedev/2019/06/18/building-hybrid-applications-with-the-webview2-developer-preview/#TQ3BjLPkJG0DgVgV.97)的测试，所以乐观估计，JS终会回到UWP中，但会基于新的渲染内核，这可能是两个大版本之后的事情了。

目前的PWA与UWP结合的事，了解即可。

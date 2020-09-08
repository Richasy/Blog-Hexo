---
title: Vue 与 Electron 开发环境的简易搭建
lang: zh
date: 2019/5/11 20:45:12
type: post
categories:
- 网页
- Vue
tags:
- vue
- electron
---

:::tip
关于借助Vue框架搭建Electron，有一个现成的解决方案，在[这里](https://github.com/SimulatedGREG/electron-vue)，可惜的是Vue和Electron的版本有些老旧
:::

<!--more-->

## 前期准备

全局安装 `vue-cli`

```sh
yarn global add @vue/cli
# or
npm install -g @vue/cli
```

了解Electron的渲染进程与主进程。

## 新建一个Vue项目

借助 `vue-cli` 可以很方便的新建一个项目，这里可以借助其可视化工具来新建。

运行

```sh
vue ui
```

![vueUI](https://cli.vuejs.org/ui-new-project.png)

关于如何新建项目，这里不表，官方说明请戳[这里](https://cli.vuejs.org/zh/guide/creating-a-project.html)


我选择的包管理是`Yarn`，构建工具是`babel`，使用`webpack`的童鞋也不必担心，原理相同，只是配置略有差别罢了。

## 对项目进行初始配置

假设你从来没有build过vue项目，那么现在，先运行：

```sh
yarn run build
```
不出意外的话，脚本会在你的项目目录中创建一个dist文件夹，现在，进到里面打开`index.html`，你会发现网页一片空白，浏览器报错说资源未找到。 

打开`index.html`的源码，你会发现所有的资源引用都引用到根目录，本地打开则会索引到磁盘根目录下。这就非我们本意了。

```html
<!DOCTYPE html>
<html lang=en>

<head>
    <meta charset=utf-8>
    <meta http-equiv=X-UA-Compatible content="IE=edge">
    <meta name=viewport content="width=device-width,initial-scale=1">
    <link rel=icon href=favicon.ico>
    <title>test</title>
    <link href=/css/app.css rel=preload as=style>
    <link href=/js/app.js rel=preload as=script>
    <link href=/js/chunk-vendors.js rel=preload as=script>
    <link href=/css/app.css rel=stylesheet>
</head>

<body>
  <noscript>
	<strong>We're sorry but test doesn't work properly without JavaScript enabled. Please enable it to continue.
	</strong>
  </noscript>
    <div id=app></div>
    <script src=/js/chunk-vendors.js> </script>
  <script src=/js/app.js> </script> 
  </body> 
</html>
```

为此，我们需要对项目进行一些简单的配置。

参照[Vue官方配置说明](https://cli.vuejs.org/zh/config/#vue-config-js)，我们在项目根目录下建立一个`vue.config.js`文件，并在其中写入：

```javascript
module.exports={
    publicPath:'./',
    // productionSourceMap:false,
    // filenameHashing:false
}

// 后两者可加可不加
```

现在再次`build`一下，你应该可以通过生成的`index.html`文件看到默认的网页了。

## 引入Electron

运行：
```sh
yarn add electron --dev
#or
npm install electron --save--dev
```

完成后，在项目根目录中建立`main.js`文件，并在其中写入：

```javascript
// Modules to control application life and create native browser window
const {app, BrowserWindow} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let mainWindow
const path=require('path');
const url=require('url');

function createWindow () {
  // Create the browser window.
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  // and load the index.html of the app.
  //mainWindow.loadURL("http://localhost:8080");
  mainWindow.loadURL(url.format({
      pathname:path.join(__dirname,'./dist/index.html'),
      protocol:'file:',
      slashes:true
  }))

  // Open the DevTools.
  // mainWindow.webContents.openDevTools()
 
  // Emitted when the window is closed.
  mainWindow.on('closed', function () {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    mainWindow = null
  })
}

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
// Some APIs can only be used after this event occurs.
app.on('ready', createWindow)

// Quit when all windows are closed.
app.on('window-all-closed', function () {
  // On macOS it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', function () {
  // On macOS it's common to re-create a window in the app when the
  // dock icon is clicked and there are no other windows open.
  if (mainWindow === null) {
    createWindow()
  }
})

// In this file you can include the rest of your app's specific main process
// code. You can also put them in separate files and require them here.

```

找到`mainWindow.loadURL()`这里，你能看到我注释了一个`localhost`的链接。由于我们这里只是简易搭建，就为了找找感觉，所以这里暂时不对开发环境和生产环境进行分别配置。

## 开发环境模拟

接上文，现在取消对`localhost`语句的注释，并把下面的`url.format()`那段语句注释掉，让我们来模拟开发环境吧。

首先，在package.json中的`scripts`中加上

=="electron":"electron ./main.js"==

加上之后的`scripts`如下所示：

```json
"scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "electron":"electron ./main.js"
  }
```

这段语句的意思是，让electron从项目根目录下的`main.js`启动。

现在打开我们的命令行工具，开两个窗口。

第一个窗口输入：

```sh
yarn run serve
```

待到启动完成，在第二个窗口中输入

```sh
yarn run electron
```

不出意外的话，你就能看到弹出了一个窗口，里面就是你的网页。

同时，它还支持热更新，文件修改后就能及时得到反馈。

## 准备打包

光看显然不行，最终electron应用都是要打包的。

目前流行的electron打包工具有两种：

[electron-builder](https://github.com/electron-userland/electron-builder)  
[electron-packager](https://github.com/electron-userland/electron-packager)

这里我们选用`electron-builder`。

运行：

```sh
yarn add electron-builder --dev
# or
npm install electron-builder --save--dev
```

使用`electron-builder`，需要在`package.json`中添加如下内容：

```json
{
  ...
  "homepage": ".",
  "main": "./main.js",
  ...
  "build": {
    "productName": "test",
    "appId": "com.richasy.test",
    "directories": {
      "output": "build"
    },
    "files": [
      "dist/**/*",
      "main.js"
    ],
    "dmg": {
      "contents": [
        {
          "x": 410,
          "y": 150,
          "type": "link",
          "path": "/Applications"
        },
        {
          "x": 130,
          "y": 150,
          "type": "file"
        }
      ]
    },
    "mac": {
      "icon": "build/icons/icon.icns"
    },
    "win": {
      "icon": "build/icons/icon.ico"
    },
    "linux": {
      "icon": "build/icons"
    }
  },
}

```

图标是可选项，没有就不写

另外`build`中的`productName`最好与根层级的`name`同名，否则也可能出现构建失败的情况。

现在，基本上准备工作就做好了，现在可以开始写构建的脚本了。

在`scripts`中加入

=="electron-build":"yarn build & electron-builder --dir"==

完成后的`scripts`如下：

```json
"scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "electron":"electron ./main.js",
    "electron-build":"yarn build & electron-builder --dir"
  }
```

这段脚本的意思就是先将当前网页打包输出到 dist 文件夹中，再通过electron-builder对其进行打包，打包完成之后，会输出到当前项目目录下的`build`文件夹中 *（此时生成的是一个文件夹，而不是安装包）* 。

现在，打开终端，运行一下吧：

```sh
yarn electron-build
```

不出意外的话，打包会成功，然后进入到`项目目录/build/win-unpacked`，找到`你的项目名.exe`，双击运行，不报错就万事大吉了。

想打成安装包的话，修改脚本如下（针对自己的目标平台进行修改）：

```json
{
  ...
  "electron-build":"yarn build & electron-builder --win --x64"
}

```
---

关于搭建Vue和Electron的简易混合开发环境的流程到这里就介绍完了。我折腾了一下午，踩了不少坑，但文中少有体现。有鉴于此，我觉得有必要把我的搭建过程写出来，让有相同需求的人少走弯路。
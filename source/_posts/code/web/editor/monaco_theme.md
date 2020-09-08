---
title: Markdown 编辑器 -3- 主题
lang: zh
date: 2020/9/2 6:52
type: post
categories:
- 网页
- 编辑器
tags:
- monaco-editor
- vue
- markdown
---

主题定义是一件比较费时的操作，尽管编辑器还是那个 VSCode 的编辑器，但如果你要假装这是自己的，那么总要在UI上有些变化才是。

这需要一些精调，所幸 monaco-editor 提供了相当丰富的主题配置选项，完全可以让你的编辑器大变样。

<!--more-->

## 相关仓库

[Monaco Themes](https://github.com/brijeshb42/monaco-themes)

## 基本说明

Monaco-editor 通过 language 解析会定义一些 token ，而语法高亮就是对这些提取出来的token进行关键词着色。

::: tip
不清楚语言解析的，可以参看[Monaco 编辑器语法定义](./monaco_language.html)，关于 token 名称，可以参看官方文档：[Monarch](https://microsoft.github.io/monaco-editor/monarch.html)
:::

主题的配置当然不止语法高亮这么简单，还涉及到其它的一些诸如编辑器背景设置，滚动条设置等，这些配置项没有文档公开，但是可以在 `editor.main.js` 中通过搜索 `----- base colors` 得到。

:::tip
`editor.main.js` 可以在 `node_modules/monaco-editor/dev/vs/editor`中找到
:::

在相同的文件中，还能找到 `vs` , `vs-dark` 和 `hc-black` 的主题定义

而创建自己主题的过程，简而言之就是复制一个主题副本，然后填色的过程。

## 根据token创建颜色配置

根据前一步[Monaco 编辑器语法定义](./monaco_language.html)得到的解析结果，我们可以创建我们自定义的颜色配置文件。

```javascript
let theme = {
  base:'vs-dark',
  inherit: true,
  rules:[
    //...
  ],
  colors:{
    //...
  }
}
```

我们的Theme对象大致就分为这几个模块，在 monaco-editor 的定义中，它的类型是 [IStandaloneThemeData](https://github.com/Microsoft/monaco-editor/blob/master/monaco.d.ts#L955)。由于 monaco-editor 在初始时就定义了三个主题，所以我们可以在此基础上进行扩展。这就是 `base` 和 `inherit` 的作用。

:::tip
选择 `vs-dark` 作为我们的基底，这意味着我们要创建一个暗色的主题（我比较喜欢），你也可以创建亮色的，那就是以 `vs` 作为基底。
:::

### Rules

在 rules 中，我们要插入的是语法高亮，根据之前的解析结果，我们可以创建一些配置，比如：

```javascript
{
  foreground: "569fff",
  token: "markdown.header",
  fontStyle: "bold",
}
```

该对象的数据结构完整定义在[这里](https://github.com/Microsoft/monaco-editor/blob/master/monaco.d.ts#L967)，

:::tip
`fontStyle`的可选项只有 `italic`, `bold` 和 `underline`  
如果你想使用多个字体效果，请使用空格隔开，比如 `fontStyle: "bold italic"`
:::

按照 `markdown.header` 的定义，我们可以把剩下来的那些 token 统统定义上。对于默认的 token，如果你不打算修改，那么可以不用写，因为我们是继承自 `vs-dark` 主题的，不写就意味着使用原有的主题色。

### Colors

这里定义的就是我之前提到的杂项设置了，比如滚动条背景，选择区域颜色之类的，大概定义如下：

```javascript
colors: {
  "editor.background": "#1E1E1E",
  "editor.foreground": "#D4D4D4",
  "editor.inactiveSelectionBackground": "#3A3D41",
  "editorIndentGuide.background": "#404040",
  "editorIndentGuide.activeBackground": "#707070",
  "editor.selectionHighlightBackground": "#ADD6FF26",
}
```

具体属性名的查找方法我也在文章开头的时候说了。

:::warning
需要注意的是，在 Rules 中定义的颜色不包含哈希字符 `#` ，而 Colors 中需要包含哈希字符。
:::

## 注册主题

主题的配置创建完成之后，我们可以在 monaco-editor 中定义属于我们的主题了：

```javascript
import * as monaco from "monaco-editor";
import theme from '../lib/markdownEx-theme.js';

monaco.editor.defineTheme("acrmd", theme);
let mdEditor = monaco.editor.create(containerDom, {
    value: "",
    theme: "acrmd",
    //...
};
```
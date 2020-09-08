---
title: Markodwn 编辑器 -1- 创建
lang: zh
date: 2020/9/1 16:50
type: post
categories:
- 网页
- 编辑器
tags:
- monaco-editor
- vue
- markdown
---

这篇博客的主要目的是用于从零开始利用monaco-editor创建一个Markdown编辑器。

由于有着相对明确的目标，所以就不会面面俱到，只能说提供一些简单的参考。

<!--more-->

## 创建方式

尽管这是第一步，但是我觉得我没什么必要把这一步写得很详细。因为我觉得官方文档和提供的示例已经足够使用了。

- [官方文档](https://github.com/Microsoft/monaco-editor)
- [代码示例](https://github.com/Microsoft/monaco-editor-samples/)

也许你在使用一些诸如Vue, React之类的框架，其实这些框架都有对应的版本，比如 [monaco-editor-vue](https://www.npmjs.com/package/monaco-editor-vue) 和 [react-monaco-editor](https://www.npmjs.com/package/react-monaco-editor)，装起来更简单。

## 在调用create方法时的一些实用参数：

文档地址：[IGlobalEditorOptions](https://microsoft.github.io/monaco-editor/api/interfaces/monaco.editor.istandaloneeditorconstructionoptions.html)

- value: 初始化的文本值
- language: 编辑器的识别语言（是代码语言而非界面语言）
- contextmenu: 是否启用默认的右键菜单
- cusorStyle: 光标样式，默认为`line`
- lineDecorationsWidth: 行号与实际内容间的距离
- lineNumbers: 是否启用行号
- lineNumbersMinChars: 行数最低字符数（占位大小），默认为5
- minimap: 对象型，其中有属性enabled，可以设置为`false`以关闭迷你地图
- mouseStyle: 鼠标指针样式
- quickSuggestions: 智能提示，可设置为false以关闭
- renderLineHighlight: 选中行的外边框，设置为false可以关闭
- scrollbar: 滚动条设置
- wordWrap: 文本自动换行，默认是不换行的，可设置为`on`
- fontSize/lineHeight等：基本的文本样式调整
- stopRenderingLineAfter: 最大渲染字符，设置为-1可以一直渲染

## 示例

```javascript
monaco.editor.create(containerDom,{
	value: [
	    '# 一个标题',
	    '> 你觉得呢？',
	    'sdasdasdasdasdad'
    ].join('\n\n'),
    language:'markdown',
    contextmenu:false, //右键菜单
    cursorStyle:'line', //光标样式
    lineDecorationsWidth:0, // 行号与实际内容间的距离
    lineNumbers:'on', //是否启动行号
    lineNumbersMinChars:1,
    minimap:{
      enabled:false
    },
    mouseStyle:'default',
    quickSuggestions:false, //智能提示
    renderLineHighlight:false, //选中行外部边框
    scrollbar:{
      arrowSize:5,
      horizontal:false,
      useShadows:false,
      vertical:true,
      verticalScrollbarSize:5
    },
    wordWrap:'on',
    fontSize:20,
    lineHeight:30,
    stopRenderingLineAfter:-1,
	}
);
```
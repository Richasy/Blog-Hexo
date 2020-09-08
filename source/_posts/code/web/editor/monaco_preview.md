---
title: Markdown 编辑器 -4- 预览
lang: zh
date: 2020/9/8 23:15
type: post
categories:
- 网页
- 编辑器
tags:
- monaco-editor
- vue
- markdown
- markdown-it
---

Markdown编辑器的**编辑**工作已经基本完成了，如果你想要查看Markdown渲染的效果，那么我这里有一些经验可以与你分享

<!--More-->

## 关于渲染器

众所周知，Markdown只不过是一个标记文本，最后渲染出来啥样，全靠渲染器和CSS。

可是CSS需要依托HTML标签，肯定无法直接渲染Markdown文本，那么就需要一个***中间商***将Markdown文本先转化为HTML文档。

市面上的渲染器有不少，但最广为人知，功能最为齐全的，当属[markdown-it](https://github.com/markdown-it/markdown-it)。

`markdown-it`本身能渲染的语法是有限的，但是通过社区插件则能极大地扩展解析范围，常见的、不常见的，统统都能解析，你所要谨慎考虑的也只有语法冲突的问题了。

## markdown-it的配置

我也不解释太多，这里分享一个我安装的markdown-it配置，包括本体及诸多插件。

本体或插件安装可以通过以下命令

```shell
yarn add markdown-it -D

#or

npm i markdown-it --save
```

配置 ( `previewConfig.js` )：

```javascript
import * as MarkdownIt from 'markdown-it';

let previewConfig = {
    getMarkdownItParser() {
        let it = new MarkdownIt({
                html: true,
                linkify: true,
                typographer: true
            }).use(require("markdown-it-container"), "tip")
            .use(require("markdown-it-container"), "warning")
            .use(require("markdown-it-container"), "danger")
            .use(require('@liradb2000/markdown-it-katex'))
            .use(require("markdown-it-underline"))
            .use(require("markdown-it-emoji"))
            .use(require("markdown-it-footnote"))
            .use(require("markdown-it-mark"))
            .use(require("markdown-it-sup"))
            .use(require("markdown-it-sub"))
            .use(require("markdown-it-ins"))
            .use(require("markdown-it-checkbox"))
            .use(require("markdown-it-abbr"))
            .use(require("markdown-it-toc-and-anchor").default, {
                anchorLink: false
            })
            .use(require("markdown-it-highlightjs"))
            .use(require("markdown-it-plantuml"))
            .use(require("markdown-it-multimd-table"))
            .use(require("markdown-it-meta"));
        let defaultRender =
            it.renderer.rules.link_open ||
            function (tokens, idx, options, env, self) {
                return self.renderToken(tokens, idx, options);
            };
        it.renderer.rules.link_open = function (
            tokens,
            idx,
            options,
            env,
            self
        ) {
            let href = tokens[idx].attrGet('href');
            if (href[0] != '#') {
                var aIndex = tokens[idx].attrIndex("target");
                if (aIndex < 0) {
                    tokens[idx].attrPush(["target", "_blank"]);
                } else {
                    tokens[idx].attrs[aIndex][1] = "_blank";
                }
            }
            return defaultRender(tokens, idx, options, env, self);
        };
        window.markdownIt = it;
        return it;
    }
}

export {
    previewConfig
}
```

markdown-it的插件是通过链式调用引入的，就像我在配置中写的那样。对于一些比较特殊的插件，比如`markdown-it-container`，还有额外的参数进行配置。

在插件引入完成后，我对链接的打开方式进行了一些调整，给外部链接添加了`target="_blank"`的属性，以避免点击链接直接在页面内打开。

## Preview模块

由于我用的是Vue，所以我直接创建一个Vue单文件即可。

### 渲染方法（示例）

```vue
<template>
  <div ref="previewContainer" id="previewContainer" v-html="html" :class="themeName"></div>
</template>


<script>
//import '../style/acrmd.css';
import "highlight.js/styles/dracula.css";
import {previewConfig} from '../lib/previewConfig.js';
//...
export default{
    name: 'preview',
    mounted() {
        window.Preview = this;
    },
    data(){
        return{
            html:'',
            markdownIt:previewConfig.getMarkdownItParser()
        }
    },

    methods:{
        markdownRender(content){
            this.html = this.markdownIt.render(content);
        }
    }
}
</script>

<style scoped>
@import url("https://cdnjs.cloudflare.com/ajax/libs/KaTeX/0.12.0/katex.min.css");
</style>
```

要是简单的调用，可以在浏览器控制台输入`window.Preview.markdownRender('# Hello Markdown!')`。

:::warning
需要注意的是，`markdown-it-highlightjs`, `markdown-it-katex`都是需要进行资源引入的，否则无法正常渲染。

你也能够在我给出的示例代码中看到我是如何引入相关资源的
:::

## 实际项目

四篇文章完成了一个简单的Markdown编辑器，由于我制作这个编辑器主要目的是为了做UWP的控件，所以在实际项目中修改了加载流程，但大体不差。

你可以在这里查看实际的项目：[Markdown-Editor-Vue](https://github.com/Richasy/Markdown-Editor-Vue)
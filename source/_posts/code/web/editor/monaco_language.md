---
title: Markdown 编辑器 -2- 语法
lang: zh
date: 2020/9/1 19:52
type: post
categories:
- 网页
- 编辑器
tags:
- monaco-editor
- vue
- markdown
---

## 官方语法定义

- [Monaco Languages](https://github.com/microsoft/monaco-languages)

- [Monarch](https://microsoft.github.io/monaco-editor/monarch.html)

## 定义语法

我也没兴趣自创语法，但默认的Markdown语法是经典语法，一些后期扩展的语法，比如删除线、上标下标、脚注等语法均未涵盖。

为了能让语法高亮得更彻底一点，必须要对当前markdown语法解析进行扩展。

<!--more-->

因为Markdown有其特殊性，在这里就无需考虑智能提示（其它语言可能会有关键词提示之类的），所以我们只需要提供三个东西。

1. 语法名
2. 解析器
3. 语法配置

语法名我们暂定为`markdownEx`，接下来解析器的构建就在原版markdown语法的基础上进行就可以了。

### 构建解析器

创建一个新的`markdownEx.js`的文件，定义一个对象`language`:

```javascript
let language = {
    defaultToken: '',
    tokenPostfix: '.md',

    // escape codes
    control: /[\\`*_\[\]{}()#+\-\.!]/,
    noncontrol: /[^\\`*_\[\]{}()#+\-\.!]/,
    escapes: /\\(?:@control)/,

    // escape codes for javascript/CSS strings
    jsescapes: /\\(?:[btnfr\\"']|[0-7][0-7]?|[0-3][0-7]{2})/,

    // non matched elements
    empty: [
        'area', 'base', 'basefont', 'br', 'col', 'frame',
        'hr', 'img', 'input', 'isindex', 'link', 'meta', 'param'
    ],

    tokenizer: {
       // paraser
    }
};
```

`tokenizer`中就是解析步骤，主要用的是正则表达式。我是正则表达式的苦手，但在已经有一堆定义的基础上，进行语法的扩展也不过就是换关键词而已。

请看下题：

已知Markdown中加粗的语法是`**Bold**`，且正则表达式为`\*\*([^\\*]|@escapes|\*(?!\*))+\*\*`，现求Mark标记语法`==Mark==`的正则表达式。

那答案就呼之欲出了：`\=\=([^\\=]|@escapes|\=(?!\=))+\=\=`。

*你不需要知道整个表达式的构成原理，只需要把关键的\\\*替换为\=即可*

所以语法的解析并不复杂，依样画葫芦罢了，如果你对Markdown究竟有哪些扩展语法感兴趣，可以查看[markdown-it](https://github.com/markdown-it/markdown-it)的说明，这个Markdown解析库应该是目前适用性最广的解析库了。

---

#### 替换token名称

在查看解析代码的时候，你可能会注意到正则表达式后面一般都跟着一个字符串，有些标注了这是token，有些没标，有些还用一个数组包起来多个token ~（由于缺少文档资料，在这里我还研究了好一会儿）~ 。

这里的token，简而言之就是你解析出来的这段特殊文本的名字，后期可以根据这个名字进行语法着色，让你的文档变得花里胡哨。

但默认的名字很难让人将markdown中的语法与token名联系起来，后面进行着色规则的配置时还要时不时回来看看，这是很累的一件事，所以在定义语法规则的时候，不妨就给它把名字重新定一下。

:::warning
这里可以重新命名是因为我只打算做markdown这一种语法，可以专门为之构建配色方案。如果你要适配多语言，切勿更改，保留原本的命名，这会让你的配色方案简单很多（因为其它的语法解析都会暴露相同的名称，不必逐个适配）
:::

比如对于引用语法，我们可以重命名如下：

```javascript
// quote
[/^\s*>+/, 'markdown.quote']
```

后期写着色规则的时候我们就可以进行匹配：

```javascript
{
    foreground: "00d2d3",
    token: "markdown.quote",
    fontStyle: "italic",
}
```

但token有一种情况比较特殊，以标题语法为例，它长这样（重命名之后）:

```javascript
// headers (with #)
[/^(\s{0,3})(#+)((?:[^\\#]|@escapes)+)((?:#+)?)/, ['white', 'markdown.header', 'markdown.header', 'markdown.header']]
```

一个数组内有多个string，这些string都是token吗？是的。

那为什么会有这么多token？为了分组。

一个完整的标题语法可以这样写：

```markdown
### Head ###
```

直观来看，它可以分成3组，前面的`#`，中间的文本与收尾的`#`，而解析器这边则分成了四组，加了行首的空格。

**这就是后面4个token的由来了。**

在正则表达式中，这种分组可以用小括号来表示，在解析时，会解析该表达式的所有`一级`小括号，这意味着三件事：

1. 表达式中的嵌套括号是不被计算在内的，只算最外层的那一级。
2. 给出的token数量如果不为一，那么要求token的数量与表达式分组的数量必须对应上
3. 不能有分组之外的单个匹配字符存在（而像`^`,`$`这样表示一种匹配模式的字符不受此规则限制）

#### 特殊语法解释

##### @语法

在解析器中，我们能看到一些比较特殊的表达式，比如

```javascript
[/^\s*\|/, '@rematch', '@table_header']
```

加了`@`符号的，表示一种引用。

`@rematch`，表示重新匹配。比如在解析自定义容器的时候，我们知道语法像这样：

```markdown
::: tip
啦啦啦啦啦啦，这是一段提示
:::
```

我们可以为三个冒号这种语法设置token，但是在解析容器内部的文本时，我们就不能一以概之的使用一种token来定义了，谁知道使用者在里面写了什么东西呢？这个时候我们就可以用`@rematch`来表示继续进行语法解析。

```javascript
//...
[/.*$/, '@rematch','@pop']
```

:::tip
关于@rematch, @pop以及其它的语法，在官方文档[Monarch](https://microsoft.github.io/monaco-editor/monarch.html)中有着详细的解释。
:::

### 构建基础配置

语法配置其实很简单，甚至可以说是通用。Markdown要配置的就是一些快捷操作：

```javascript
let config = {
    comments: {
        blockComment: ['<!--', '-->',]
    },
    brackets: [
        ['{', '}'],
        ['[', ']'],
        ['(', ')'],
    ],
    autoClosingPairs: [
        { open: '{', close: '}' },
        { open: '[', close: ']' },
        { open: '(', close: ')' },
        { open: '<', close: '>', notIn: ['string'] },
        { open: '`', close: '`', notIn: ['string'] },
        { open: '```', close: '\n\n```', notIn: ['string'] },
        { open: '（', close: '）' },
        { open: '【', close: '】' },
        { open: '《', close: '》' },
        { open: '“', close: '”' },
        { open: '‘', close: '’' },
        { open: '\'', close: '\'' },
        { open: '"', close: '"' },
    ],
    surroundingPairs: [
        { open: '(', close: ')' },
        { open: '[', close: ']' },
        { open: '`', close: '`' },
    ],
    folding: {
        markers: {
            start: new RegExp("^\\s*<!--\\s*#?region\\b.*-->"),
            end: new RegExp("^\\s*<!--\\s*#?endregion\\b.*-->")
        }
    }
}
```

这里的notIn，在monaco的源码中也语焉不详，我也不知道可选值有哪些，只能将就着先用着。

相比起默认的配置，我对中文中的全角符号进行了一些适配，比如书名号，引号之类的。

这一块没啥好说的，各个模块的说明可以参见文档：[LanguageConfiguration](https://microsoft.github.io/monaco-editor/api/interfaces/monaco.languages.languageconfiguration.html#autoclosingpairs).

## 使用语法

先将我们之前定义的解析器和配置模块导出：

```javascript
export { language, config };
```

到这一步就还挺简单的，因为我们只需要定义一个解析器和一个语法配置器，所以可以这样写：

```javascript
import { language, config } from "../lib/markdownEx.js";
import * as monaco from "monaco-editor";

monaco.languages.register({ id: "markdownEx" });
monaco.languages.setMonarchTokensProvider("markdownEx", language);
monaco.languages.setLanguageConfiguration("markdownEx", config);
let mdEditor = monaco.editor.create(containerDom, {
    value: "",
    language: "markdownEx",
    //...
};
```
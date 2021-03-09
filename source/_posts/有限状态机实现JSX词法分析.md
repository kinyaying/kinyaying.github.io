---
title: 有限状态机实现JSX词法分析
date: 2021-02-09 13:58:06
tags: node
keywords:
  - jsx词法分析
  - 优先状态机
  - node
---

词法分析和语法分析是代码中非常底层的知识，在代码的编译、构建的过程中扮演了重要的角色。
词法分析是指把代码片段逐个单词的读取，根据分词规则转化成一个个 token。每个 token 有对这段单词的类型和内容进行描述。最后将这些 token 收集成 tokens 返回回来。目前常用的 jsx 词法解析器是[esprima](https://github.com/jquery/esprima/blob/master/src/jsx-parser.ts) 。

为了掌握 jsx 词法解析，我也实现了简易版的 jsx 词法解析器。

**代码仓库：[jsx-tokenizer](https://github.com/kinyaying/tokenizer)**

**npm: [jsx-tokenizer](https://www.npmjs.com/package/jsx-tokenizer)**

<!-- more -->

### 有限状态机

有限状态机，（英语：Finite-state machine, FSM），又称有限状态自动机，简称状态机，是表示有限个状态以及在这些状态之间的转移和动作等行为的数学模型。

状态机有以下特点：

- 每一个状态都是一个机器,每个机器都可以接收输入和计算输出
- 机器本身没有状态,每一个机器会根据输入决定下一个状态

##### 优点：在状态比较多的情况下，把状态、事件及 transition 集中到一个状态机中，进行统一管理。这样不需要写太多的 if-else，或者 case 判断，如果增加状态和事件，也便于代码的维护和扩展。

### JSX 词法分析

词法分析概念：接收原始代码,然后把它分割成一些被称为  token  的东西，这个过程是在词法分析器(Tokenizer 或者 Lexer)中完成的。

目前常用的库有：[esprima](https://github.com/jquery/esprima/blob/master/src/jsx-parser.ts) ，可以将 JSX 语法转换成抽象语法树。

```js
let esprima = require('esprima')
let sourceCode = 'let dom = <h1>hello world</h1>'
let ast = esprima.parseModule(sourceCode, { jsx: true, tokens: true })
```

上代码的运行结果如下，例如：`let dom = <h1>hello world</h1>` 这个代码片段被分割成 tokens 里的片段。

```
Module {
  type: 'Program',
  body: [
    VariableDeclaration {
      type: 'VariableDeclaration',
      declarations: [Array],
      kind: 'let'
    }
  ],
  sourceType: 'module',
  tokens: [
    { type: 'Keyword', value: 'let' },
    { type: 'Identifier', value: 'dom' },
    { type: 'Punctuator', value: '=' },
    { type: 'Punctuator', value: '<' },
    { type: 'JSXIdentifier', value: 'h1' },
    { type: 'Punctuator', value: '>' },
    { type: 'JSXText', value: 'hello world' },
    { type: 'Punctuator', value: '<' },
    { type: 'Punctuator', value: '/' },
    { type: 'JSXIdentifier', value: 'h1' },
    { type: 'Punctuator', value: '>' }
  ]
}
```

esprima 内部就是使用了有限状态机的思想，逐步分析 jsx 语句中的每个字符，并将结果组装成对应的 node，收集到 tokens 里。

### 自己实现 JSX 词法解析

我们也可以自己实现词法解析，将 jsx 语句转换成 tokens。我自己写的方法和 esprima 相比扩展性稍弱，且考虑的场景也相对简单，便于让人理解状态机模式以及使用在词法分析场景下的应用。

举个例子，处理 sourceCode='<span>hello</span>'这段代码，会依次识别每个字符，根据当前字符推断下一个运行的函数是什么。流程图如下：

![图1. 词法解析流程图](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fd999474d88347328df7847637ddc275~tplv-k3u1fbpfcp-zoom-1.image)

我将写的[jsx-tokenizer](https://www.npmjs.com/package/jsx-tokenizer)放到了 npm 上，可以 down 下来跑一下；也可以在[github 仓库](https://github.com/kinyaying/tokenizer)里查看。

**使用步骤：**

1. 安装

```shell
npm i jsx-tokenizer
```

2. 使用

```js
const { tokenizer } = require('jsx-tokenizer')
let sourceCode =
  '<h1 id="title" style="background: green;"><span>hello</span>world</h1>'
console.log('tokenizer词法解析结果:', tokenizer(sourceCode))
```

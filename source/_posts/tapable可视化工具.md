---
title: tapable可视化工具
date: 2021-02-09 13:58:06
tags: react
keywords:
  - tapable可视化编辑器
  - umi
  - mxGraph
  - esprima
  - monaco-editor
---

# 功能介绍

webpack 中的发布订阅模式是使用 `tapable` 实现的。`tapable` 比 node 自带的 `evevnEmitter` 功能更加强大，表现在 1. 一次发布能触发多个订阅钩子； 2. 事件流种类更多，分为 bail，basic, loop, waterfall。当然，学习难度也更高。我开发这个工具的目的是想将 tabable 代码转换成流程图，更直观的了解对应的 `tabable` 执行流程。

<!-- more -->

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16e36490ff5047579d782bf89c4774cf~tplv-k3u1fbpfcp-watermark.image)

**思路：**

1. 获取左侧代码片段
2. 将代码片段经过 ast 语法树解析，解析出使用的 tapable 钩子

3. 针对不同的钩子渲染不同的流程图

**有了思路我变开始选择使用的库：**

- mxGraph： 绘制流程图
- esprima + estraverse: 解析分析代码
- monaco-editor：代码编辑器
- Umi：基础框架

当初设想的以为很简单，但是实现起来发现用到的每个库挺大，有一定的学习难度。所以这个项目是试水项目，我也算基本对这几个库入了门，后期有需要再进行深入学习。

页面地址：[tapable 可视化工具](https://kinyaying.github.io/tapableUtil/dist/index.html)

github 地址：[tapableUtil](https://github.com/kinyaying/tapableUtil)

# 编辑器（monaco-editor ）

**核心代码：**

```js
// 1. 初始化编辑器
let monacoInstance = monaco.editor.create(document.getElementById('editor'), {
  theme: 'vs-dark',
  scrollBeyondLastLine: false,
  value: '代码片段',
  language: 'javascript',
})
// 2. 获取代码片段
let code = monacoInstance.getValue()
```

**需要注意的坑：**

代码中如果只引入` monaco-editor` 会发现界面是出来了，但是没有语法高亮。在控制台输入 `monaco.languages.getLanguages()`发现根本没有 javascript 语法，只有一个最基础的 plaintext。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b6aedc831e24971a32ca4c85c2eb446~tplv-k3u1fbpfcp-watermark.image)

此处有两种解决方案：

1. 代码中引入 `monaco` 自身提供的 javascript 内置语言

   ```js
   import 'monaco-editor/esm/vs/basic-languages/javascript/javascript.contribution'
   ```

2. 修改 webpack 配置，如果使用的是 umi，也可以按照 umi 官网有提供配置方法([monaco-editor 编辑器打包](https://umijs.org/zh-CN/guide/boost-compile-speed#monaco-editor-%E7%BC%96%E8%BE%91%E5%99%A8%E6%89%93%E5%8C%85))。

   PS. umi 的 webapck 配置文件需在根目录下新建 config/config.js 文件。

# 代码解析（esprima + estraverse）

esprima: 一个强大的库，将 js 代码转 ast 语法树

estraverse: 提供遍历 ast 语法树的钩子

[astexplorer](https://astexplorer.net/): 提供可视化的代码片段转 ast 语法树网站，遍历 ast 语法树时对应此网站的结构能更方便的知道自己想获取的节点结构

**核心代码：**

```js
// 代码转 ast
let ast = esprima.parse(code)
// 遍历 ast 语法树
estraverse.traverse(ast, {
  enter(node) {
    // xxx 具体代码
  },
})
```

具体的代码可以参考 `esprima` 和 `estraverse` 的官方文档，此处不再具体介绍。在遍历语法树时，可以根据 [astexplorer](https://astexplorer.net/) 这个网页展示的语法树结构进行遍历。

# 图像渲染（mxGraph）

**核心代码：**

```js
var graph = new mxGraph(container)
// 清空画布
graph.removeCells(graph.getChildVertices(graph.getDefaultParent()))
// 开始绘制
graph.getModel().beginUpdate()
// 绘制图形
var cell = graph.insertVertex(parent, 0, tap, x, y, w, h)
// 绘制 cell 和 step 之间的连线
graph.insertEdge(parent, null, '', cell, step)
```

**需要注意的点：**

1. 交互上涉及到点击按钮后重新绘制流程图，所以需要在重新绘制前清空图层中的单元。否则会出现新的图块和旧图块混在一起的情况。

```js
graph.removeCells(graph.getChildVertices(graph.getDefaultParent()))
```

2. 连线时为了避免 ① 和 ② 两条线黏连在一起，设置了线上的点的位置

   ```js
   var line = graph.insertEdge(parent, null, 'false', step, end)
   // 固定线经过的点的位置
   line.geometry.points = [new mxPoint(pointX, pointY)]
   ```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06757a12eca64bba8b017dbb42e7108d~tplv-k3u1fbpfcp-watermark.image)

# 参考

[mxGraph 官方文档](https://jgraph.github.io/mxgraph/docs/manual.html)
[mxGraph 的各种 demo](http://jgraph.github.io/mxgraph/javascript/index.html)
[Monaco Editor API](https://microsoft.github.io/monaco-editor/api/modules/monaco.editor.html)
[umi 官网](https://umijs.org/zh-CN/docs)
[正则可视化工具](<https://jex.im/regulex/#!flags=&re=%5E(a%7Cb)*%3F%24>)

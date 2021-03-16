---
title: 开发油猴脚手架wokoo复盘
date: 2021-02-12 13:58:06
tags: node
keywords:
  - 手写脚手架
  - nodejs
  - 油猴插件
---

# 一、项目简介：

这篇文件介绍的是油猴插件脚手架Wokoo的开发过程与复盘。Wokoo是我开发的脚手架，用来快速起一个基础项目用于`油猴插件`的开发。

1. Wokoo的npm地址：[wokoo](https://www.npmjs.com/package/wokoo)
2. Wokoo的github地址：[wokoo](https://github.com/kinyaying/wokoo)  (如果觉得不错，请点小星星)
3. Wokoo脚手架开发的油猴插件有：
   - [知乎目录](https://greasyfork.org/zh-CN/scripts/421238-zhihu-helper)
   - [划词搜索](https://greasyfork.org/zh-CN/scripts/421189-movesearch)
4. Wokoo使用说明文档 👉 [5分钟上手开发浏览器插件——油猴脚手架wokoo(使用篇)](https://juejin.cn/post/6922815205575491597)
5. 不了解什么是油猴插件？戳这里 👉  [Chrome 插件大杀器：「油猴」Tampermonkey 使用详解](https://zhuanlan.zhihu.com/p/99390731)

在这个项目中我扮演了项目的`产品经理`，`开发主R`，`运营官` 这几个角色。从头到尾负责开发一个开源项目，让我有了满满的成就感。
<!-- more -->
回想当初想做Wokoo脚手架的初衷，只是因为自己想写一个油猴插件，发现比较麻烦，自己也趟了一些坑。我就觉得写一个脚手架，能够一键生成基础的油猴项目很有意义。完成Wokoo脚手架的开发后，我在某技术群中分享了这篇文章，被该圈里的大佬邀请做一次分享。于是我又增加了[使用Wokoo进行项目实战的案例](https://juejin.cn/post/6925605904561750030)。然后又指导群里成员完成从零上手，实现一个油猴插件的开发。

整个过程在当初只是一个小小的idea，在逐步升级打怪的过程中，我把当初的小想法变成了一个正式的开源项目。这次经历让我得到很多成长，包括技术、沟通、合作等方面。感谢这次经历~ 🎉

# 二、项目背景：

## 开发Wokoo脚手架初衷

一开始我想做一个给团队内部使用的浏览器插件。进行技术选型时发现开发油猴插件开发最省时省力，而且不用审核上线迅速，是最佳选择。

开发过程遇到一些坑：

1. 油猴插件本质上是嵌入一段js代码到当前html中。所以很多油猴插件都直接使用jQuery进行开发。这对于我们习惯使用了Vue、React等框架的rd来说会不适应，尤其是插件涉及到页面组件开发，使用jQuery有点痛苦。**这就是第一个痛点：要配置vue或react基础项目**

2. 所以我就考虑用vue-cli起一个vue项目来完成插件开发。项目创建完成后，我又发现要配置油猴脚本。下面这个编辑器里的代码就是要配置的油猴脚本，里面的注释都是有含义的，比如`@match`  指定某些域名下开启此插件，`// @match http://*/*`  表示只能在http协议的网页中使用。默认配置的就是http协议，但是现在的网页都升级到https了，如果不修改此条配置，你会发现插件根本跑不起来。👇

![屏幕快照 2021-03-15 下午9.30.21.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/546d2d1512d04635b77e4db04d267ad4~tplv-k3u1fbpfcp-watermark.image)

   **第二个痛点：要阅读油猴脚本的文档，理解里面的注释内容**

3. 使用vue开发的项目能够通过`localhost:8080`访问，但是怎么把项目和油猴插件结合起来，实现一边开发项目，一边实时调试插件？总不能一边开发，一边`npm run build`构建出包，再把构建结果复制到油猴插件的编辑器中吧？**第三个痛点：如何实现边开发边展示插件效果？**

## 业务场景

总结下，我想实现一个针对开发油猴插件的脚手架，满足如下功能：

1. 输入一个命令行生成一个基础项目
2. 项目中包含基础的油猴脚本配置
3. 提供方案，让油猴插件能边开发，边调试

## 解决的痛点
wokoo脚手架就是为了满足上业务场景，解决我在开发油猴插件时遇到的3个痛点。
1. 一键式创建基础的油猴插件项目，可选模板有:

   [ ] vue

   [ ] react


2. 基础项目中的tampermonkey.js文件中满足油猴插件的基础配置，例如：

   ```js
   // ==UserScript==
   // @name         my-plugin
   // @namespace    http://tampermonkey.net/
   // @version      0.0.1
   // @description  try to take over the world!
   // @author
   // @match        https://*/*
   // @match        http://*/*
   
   // ==/UserScript==
   
   ;(function () {
     'use strict'
     if (location.href === 'http://localhost:8080/') return
     var script = document.createElement('script')
     script.src = 'http://localhost:8080/app.bundle.js'
     document.body.appendChild(script)
   })()
   ```

   开发者直接复制这段代码到油猴插件编辑器就行，解决上面的第二个痛点。

   当然开发者在开发过程中是要做到阅读 [tampermonkey官网文档 ](https://www.tampermonkey.net/documentation.php?version=4.6&ext=dhdg)的，wokoo脚手架提供的基础文档配置，目的是降低油猴插件的开发难度。

3. tampermonkey.js中写的js脚本，将油猴插件加载的js文件打到本地用webpack起的服务上，实现边开发边调试。解决上面的第三个痛点。

# 三、实践过程：

## 技术选型

- 参考了[create-react-app](https://github.com/facebook/create-react-app)的设计思路，将`wokoo-scripts`和`wokoo-template`部分解耦，拆分成两个包单独管理，有利于未来的功能拓展。比如未来要新增一个模板是基于jquery的，只需要拓展wokoo-template部分即可。
- monorepo方案管理代码，使用lerna进行包管理。

## 开发过程

这张图是Wokoo的架构设计图，Wokoo使用monorepo方案管理代码，在一个git仓库中维护`wokoo-scripts`和`wokoo-template`两个模块。使用lerna进行包管理。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf77f0701b274c7483a0d6aaba207640~tplv-k3u1fbpfcp-watermark.image?imageslim)

## 思路整理

我要实现的功能如下：

1. 安装wokoo，执行`npm i wokoo -g`后在npm的全局目录下安装wokoo

2. 执行命令`wokoo my-app`，弹出要求用户选择模板的提示

   ```
   ? which template do you prefer? (Use arrow keys)
   ❯ vue
     react
   ```

3. 选择模板后，创建一个基础项目，项目内容包括：1. 配置好的vue或react项目，2. `tampermonkey.js` 文件，内部是配置好的基础油猴脚本。

模仿[cra](https://github.com/facebook/create-react-app)的实现思路，我将wokoo拆分成两个部分，`wokoo-scripts`和`wokoo-template`。

- `wokoo-scripts`负责命令行的交互，从npm上拉取`wokoo-template`到生成的基础项目my-app中。
- `wokoo-template`提供两套模板，分别是轻量级配置的vue和react基础项目。还提供`tampermonkey.js` 文件

整个项目的工作流入下👇：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/666e725ea331445387a7f8ac8f76b106~tplv-k3u1fbpfcp-watermark.image)

### wokoo-scripts

目录介绍：

```
.
├── README.md
├── bin
│   └── www    					提供给npm的入口文件
├── index.js						主程序，提供命令行交互功能
├── modifyTemplate.js 	替换模板中的ejs语法
├── package-lock.json
└── package.json
```

代码流程图：


![流程图.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6b207eeeac1459889c3094468bc0892~tplv-k3u1fbpfcp-watermark.image)

图中介绍了`wokoo-scripts`各个方法的调用顺序： init -> createApp ->run->modifyTemplate。以及各方法实现的功能。

### wokoo-template

目录介绍：

```
.
├── README.md
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   ├── icon.jpg
│   └── index.html							html文件
├── react-template							react模板路径
│   ├── README.md
│   ├── src
│   │   ├── app.js							
│   │   ├── app.less
│   │   └── index.js						入口文件
│   ├── tampermonkey.js					油猴脚本配置
│   ├── template.json						react中依赖的库文件
│   └── webpack.config.base.js	对应react项目的webpack配置
├── vue-template								和react的项目结构类似
│   ├── ...				
└── webpack.config.js						公共的webpack配置
```

`wokoo-template`提供基础项目的模板：

- 分为vue-template和react-template

- vue-template和react-template分别对应webpack配置的一个vue或react基础项目

- 使用ejs模板，实现wokoo-scripts注入变量

- tampermonkey.js 文件是油猴插件的配置文件，需要将此文件内的代码复制到油猴插件的编辑框中。如下是tampermonkey.js文件的内容：

  ```js
  // ==UserScript==
  // @name         <%=projectName%>
  // @namespace    http://tampermonkey.net/
  // @version      <%=version%>
  // @description  try to take over the world!
  // @author
  // @match        https://*/*
  // @match        http://*/*
  
  // ==/UserScript==
  
  ;(function () {
    'use strict'
    if (location.href === 'http://localhost:8080/') return
    var script = document.createElement('script')
    script.src = 'http://localhost:8080/app.bundle.js'
    document.body.appendChild(script)
  })()
  
  ```

此文件中被注释掉的`//@xxx `都有含义，可以对应着 [tampermonkey开发文档 ](https://www.tampermonkey.net/documentation.php?version=4.6&ext=dhdg) 理解。

- `@name`  脚本的名字，最后会被替换成项目名
- `@namespace` 可以写自己的域名,当自己把脚本分享后,用户可以直接通过这儿找到你的具体功能实现

- `@description`  插件描述
- `@match`  指定某些域名下开启此插件，默认配了两条，`// @match        https://*/*`和`// @match        https://*/*`  表示在所有域名下都开启。

详细的开发过程可以阅读文章：[wokoo脚手架（搭建篇）](https://juejin.cn/post/6925613440752943112)

## 成果展示

1. 创建基础项目

   ```shell
   npx wokoo my-app
   ```

2. 起服务

   ```
   npm start
   ```

3. 复制tampermonkey.js内容到[油猴插件编辑器](chrome-extension://dhdgffkkebhmkfjojejmpbldmpobfkfo/options.html#url=&nav=new-user-script)，注意：要复制全部内容，包括注释部分。（此步骤默认你已经安装了油猴插件，没安装的话就安装下 👉[油猴插件安装地址](https://chrome.google.com/webstore/detail/tampermonkey-beta/gcalenpjmijncebpfijmoaglllgpjagf?hl=zh-CN)）


![屏幕快照 2021-03-15 下午9.30.21.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/811cad2db5cb4332bfec83ea697aa2a2~tplv-k3u1fbpfcp-watermark.image)

4. 打开某网页，你能看到一只的猴子🐒，代表流程已跑通，你只需开发自己的业务代码即可。

![demo2.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f8e6a5a044f4f2980febedf066bbcd1~tplv-k3u1fbpfcp-watermark.image)

## 开发中遇到的问题及解决方案

**问题1**

油猴插件的本质是往页面中注入一段js脚本，要注意避免和原始网页之间的冲突。拿知乎举例子，

1. 使用wokoo脚手架创建的项目的根节点id = root ，而知乎根节点网页中已有一个div节点id=root，命名重复了。
2. 不光根节点id的命名要注意避免和宿主网页中的元素重复，还要wokoo创建的项目A和项目B的根节点id不同。因为用户可能同时使用多个油猴插件，比如「划词搜索」和「知乎目录」一起用，这就要确保两个插件的根节点id是不一样的。

**解决方式**

将wokoo-template中的根节点根据`wokooApp-${项目名}-${时间戳}`的规则进行命名，避免冲突。

**问题2**

有的网页安全策略做的比较好，比如知乎使用了csp内容安全策略，阻止加载非指定域名的js脚本。

结合实际情况来看，比如我们已经使用wokoo初始化一个项目my-app，并完成了[油猴脚本](chrome-extension://dhdgffkkebhmkfjojejmpbldmpobfkfo/options.html#url=&nav=new-user-script)的配置，打开知乎发现右上角没有出现猴子logo，并且控制台报错。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1c6afc5b4cb4494b72c0d7b685f5579~tplv-k3u1fbpfcp-zoom-1.image)

再看请求的html资源的响应头，可以发现多了一条`content-security-policy`规则。也就是说知乎使用了csp内容安全策略，通过`content-security-policy`中的`script-src`字段可知，知乎只允许加载指定域名的js。具体情况可阅读 👉 [内容安全策略( CSP )](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a7ce3b03a5434e2aac0a911940d01a0d~tplv-k3u1fbpfcp-zoom-1.image)

而将tampermonkey.js文件拷贝到[油猴插件编辑器](chrome-extension://dhdgffkkebhmkfjojejmpbldmpobfkfo/options.html#url=&nav=new-user-script)时，有下面一段代码👇

```js
// ==/UserScript==
;(function () {
  'use strict'
  if (location.href === 'http://localhost:8080/') return
  var script = document.createElement('script')
  script.src = 'http://localhost:8080/app.bundle.js'
  document.body.appendChild(script)
})()

```

因为我们在调试的时候为了保证实时查看插件效果，往页面的html拼了一个js文件上去，而该文件指向webpack起的服务：`http://localhost:8080/app.bundle.js`。这不是知乎指定的域名，当然被拦截了。

**解决方式**

怎么办呢？嘿嘿，魔高一尺道高一丈，wokoo脚手架当然给出了解决方案。

1. 开发阶段，安装插件[Disable Content-Security-Policy](https://chrome.pictureknow.com/extension?id=5b80153e8db143afa59310bc0f282f1f), 在调试知乎页面时开启插件，自动把html页面的`content-security-policy`给设置为空。

   ![截图](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba9ac0c0e6d14367bcd51b3aff3a9e2b~tplv-k3u1fbpfcp-zoom-1.image)

2. 上线到[油猴市场](https://greasyfork.org/)时，将构建后的脚本复制到油猴插件编辑器中，避免使用cdn的方式部署。

   ![zhihu-tampermonkey](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5e2da6218f824bc7917a43b107362899~tplv-k3u1fbpfcp-zoom-1.image)

- 注意，编辑框内对代码有最大限制，如果app.bundle.js大小超过最大限制，要进行拆包处理。

  拆包步骤：

  - 油猴编辑器内的配置代码，增加两行。此处我用react举例，把react通过静态资源链接的方式引入

    ```
    // @require https://unpkg.com/react@17/umd/react.production.min.js
    // @require https://unpkg.com/react-dom@17/umd/react-dom.production.min.js
    ```

  - 修改webpack.config.base.js的entry字段

    ```
    entry: {
        app: '/src/index.js',
        vendor: [
          // 将react和react-dom这些单独打包出来，减小打包文件体积
          'react',
          'react-dom',
        ],
      }
    ```

  - 重新构建代码，把代码复制到编辑框中。

具体的wokoo使用方式我有写专门的文档：

[5分钟上手开发浏览器插件——油猴脚手架wokoo(使用篇)](https://juejin.cn/post/6922815205575491597)

[快速上手油猴插件开发（实战篇）](https://juejin.cn/post/6925605904561750030/)

# 四、总结思考：

## 技术上收获

1. 为了把脚手架写的专业一点，我去研究了create-react-app的源码，看大佬写的代码真的是佩服，自己以后在写代码时也要多考虑组织结构，逻辑解耦。
2. 使用monorepo方案，一个git仓库管理多个库文件（包括[wokoo-script](https://www.npmjs.com/package/wokoo)和[wokoo-template](https://www.npmjs.com/package/wokoon-template)，发现真香👍。 解决了开发多个相关联的库时，需要切换不同库的仓库，代码等问题，保证工作流的连续。

总之，在搞技术的道路上还是一句话：「纸上得来终觉浅，绝知此事要躬行」。

## 思维上收获

1. 我意识到做一件事情，小到一次分享，一篇文章，大到一个开源项目，一个产品，最根本的出发点应该是「更好的服务产品的使用方」。只有服务好使用方，你做的事情才能发挥最大价值。

   > 比如当初在群里分享利用Wokoo脚手架开发插件「划词搜索」时，发现总是有人问我一些简单的问题，我当时就想文档里明明写了啊，后来再次读那段内容时，我发现我是站在开发者的角度去写文档，而没有站在使用者的角度写。

2. 任何事情，一开始看可能觉得难度巨大，但是拆解成一件件小的事情后就会觉得不是那么困难。就像`Wokoo脚手架`，也是被拆分成多个步骤完成：

   - 开发脚手架
   - 编写使用文档
   - 写油猴插件实战案例

   所以遇到事情先拆解，拆分成几个小步骤来完成。不要一看到大项目就畏难就觉得搞不定。


# 五、往期文章

除了油猴脚手架wokoo，我还造了其他轮子🎉🎉：

- [shero-cli: 自动化管理github博客](https://juejin.cn/post/6917256394743742472)

- [tapable可视化工具](https://juejin.cn/post/6907563804780036104)

欢迎大家在搞技术的路上一起升级打怪造轮子~

——

感谢你的阅读😊，希望这篇文章对你有所帮助。
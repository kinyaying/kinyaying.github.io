# 5分钟上手开发浏览器插件——油猴脚手架wokoo(使用篇)

## 油猴插件是什么？

油猴插件(Tampermonkey) 是一款免费的浏览器扩展和最为流行的用户脚本管理器，它适用于 Chrome, Edge, Safari等多个浏览器。油猴脚本本质上是在网页上插入一段 JavaScript(JS) 代码，开发者在代码中编写内容，开发插件。此外，它还提供Userscript Header和Application Programming Interface给开发者，用来实现原生 JS 无法实现的功能。

油猴插件的开发文档请看这里 👉 [[油猴插件开发文档](https://www.tampermonkey.net/documentation.php?version=4.6&ext=dhdg)]

## 为什么要开发wokoo脚手架

油猴插件的基本原理是支持用户使用js编写脚本代码，再在网页的环境上下文运行。

在油猴插件生成的基础项目中，可以看到什么都没有配置，它只支持原生js开发。这对于我们熟悉vue或react的FEer来说影响效率。我们需要自己搭建一个基础的项目工程，进行开发。还要配置油猴脚本内容，确保油猴插件引入了我们开发的js代码，比较麻烦。而wokoo则是为了解决这个问题。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88f5eca70aa145feb6ca36b90c04665d~tplv-k3u1fbpfcp-watermark.image)

先介绍下wokoo脚手架的成果：

[wokoo脚手架github仓库](https://github.com/kinyaying/wokoo)

安装油猴插件：[油猴插件安装地址](https://chrome.google.com/webstore/detail/tampermonkey-beta/gcalenpjmijncebpfijmoaglllgpjagf?hl=zh-CN)

安装demo插件: [wokoo-demo](https://greasyfork.org/zh-CN/scripts/420327-wokoo-demo) （详细安装过程可查看 ➡️ [wokoo-demo/readme](https://github.com/kinyaying/wokoo/tree/master/example/wokoo-demo)）

安装完成后的效果如下：

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e978d446a66435b9cf1f9b30eee979d~tplv-k3u1fbpfcp-watermark.image)

## wokoo是什么

一款油猴插件的脚手架。如果直接开发油猴插件，开发者需要费时搭建vue或react基础项目，还需要对油猴脚本区域做对应的配置，开发体验差。

wokoo可以一键式生成基础项目，并且提供基础Tampermonkey配置。

主要提供的功能有：

- 命令行式创建脚手架初始项目
- 根据用户选择，生成vue、react的基本项目
- tampermonkey.txt 文件中提供Tampermonkey配置

## 项目设计图

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cf77f0701b274c7483a0d6aaba207640~tplv-k3u1fbpfcp-watermark.image)

wokoo脚手架的设计参考了[create-react-app](https://github.com/facebook/create-react-app)，我也曾经写过一篇分析cra源码的文章，感兴趣的同学可以阅读这篇👉[create-react-app核心源码解读](https://juejin.cn/post/6916531902773985294)。

wokoo主要使用lerna进行包管理，在packages里维护了两个模块：wokoo-scripts 和 wokoo-template。

**wokoo-scripts**：

分析终端用户输入的命令行，根据用户输入的选项生成对应的初始化项目

**wokoo-template**：项目模板

- 分为vue和react
- 支持基础的webpack配置
- tampermonkey.txt 文件中是油猴编辑器内容，用户不必自己编写油猴脚本的配置。并且此处做了优化，在tampermonkey.txt中使用动态引入js脚本的方案，而不是@require引入js文件方式，解决开发时静态资源缓存问题。

wokoo工作流如下图👇：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/666e725ea331445387a7f8ac8f76b106~tplv-k3u1fbpfcp-watermark.image)

## wokoo脚手架使用

1. 安装 

```
npm i wokoo -g
复制代码
```

2.  创建my-plugin项目

```
wokoo my-plugin
复制代码
```

3. 选择框架：控制台出现提示 ➡️ which template do you prefer?

[ ] vue

[ ] react

4.  选择完毕后自动初始化项目

5. cd my-plugin 能够看到生成的原始项目

## 目录结构

```
.
├── README.md 
├── package-lock.json
├── package.json
├── public                    静态文件
│   ├── favicon.ico
│   ├── icon.jpg
│   └── index.html            html文件
├── src
│   ├── app.less
│   ├── app.js
│   └── index.js             项目入口
├── tampermonkey.txt         油猴脚本入口文件
├── webpack.config.base.js
└── webpack.config.js        webpack 配置
复制代码
```

此处展示的是react项目的目录结构，vue项目的结构类似，不再展示。

其中`tampermonkey.txt`文件内是油猴脚本配置，需要将里面的内容复制到Tampermonkey编辑器里。

## 开发&调试

1. 启动 

进入项目目录后，在命令行中输入：

```
npm start
复制代码
```

2. 打开浏览器，输入`localhost:8080`，查看页面展示是否正常。

3. 打开油猴插件编辑界面，将 `tampermonkey.txt` 里的内容复制到编辑框中，保存。（此步骤默认你已经安装了油猴插件，没安装的话就安装下 👉[油猴插件安装地址](https://chrome.google.com/webstore/detail/tampermonkey-beta/gcalenpjmijncebpfijmoaglllgpjagf?hl=zh-CN)）

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/05cb9892e1bd42f4bd12e36f39e185a2~tplv-k3u1fbpfcp-watermark.image)

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3160f3a1c95147ed8fd8d598b9b1966d~tplv-k3u1fbpfcp-watermark.image)

`tampermonkey.txt` 中的js逻辑是给html拼上script标签来获取外部js文件，在开发过程中不要担心缓存问题，开发完代码后能直接在浏览器看到最新的效果。

4. 打开任意一个网页，比如[www.baidu.com](http://www.baidu.com/)

- 查看油猴 icon 是否有一个 1 的数字标志，如果有说明油猴脚本已经成功激活
- 网页的右上角会出现一只猴子，说明代码已经跑通

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4304f711784a440db86d03f237f0b81a~tplv-k3u1fbpfcp-watermark.image)

5. 接下来可以修改业务代码进行自定义的插件开发啦~ 🎊🎊

## 构建

执行命令

```
npm run build
复制代码
```

## 发布插件到油猴市场

发布油猴市场的优点是不用审核，即发即用，非常方便。

1. 将`/dist/app.bundle.js` 文件部署到 cdn 上，获取到对应 url。

注意：

- js文件可放到 github 上，如果托管到 github 上最好做 cdn 加速

- 如果没有cdn服务器可跳过此步骤，在步骤4直接将app.bundle.js复制到油猴脚本编辑器中

2. 登录[油猴市场](https://greasyfork.org/)，谷歌账号或 github 账号都可使用。

3. 点击账号名称，再点击「发布你编写的脚本」![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abcdfbc992fb4fd69a2178ca94876071~tplv-k3u1fbpfcp-watermark.image)

   ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb025dd64bc643a48858a875f9c70fdc~tplv-k3u1fbpfcp-watermark.image)

4. 进入编辑页，将 `tampermonkey.txt` 里的内容复制到编辑框中

注意：

- 步骤1中如果托管了cdn，需要将代码中的`localhost:8080`网址替换成静态资源 url

- 步骤1中没有托管cdn，则将`/dist/app.bundle.js`文件里的内容复制到下图红框位置

  ![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7aaced8f163842af9e165b7a98388198~tplv-k3u1fbpfcp-watermark.image)

5. 点击 「发布脚本」即可

## 参考

[create-react-app核心源码解读](https://juejin.cn/post/6916531902773985294)

[油猴市场](https://sleazyfork.org/zh-CN/scripts?page=1) 

[油猴插件开发文档](https://www.tampermonkey.net/documentation.php?version=4.6&ext=dhdg)

——

未来wokoo脚手架也会持续更新，如果有相关问题或建议可以在github上提issue或者联系我 email: [kinyaying@gamil.com](mailto:kinyaying@gamil.com)。
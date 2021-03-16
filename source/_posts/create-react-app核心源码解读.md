![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da2ddd71d01541f8a8c78bf01d9e249c~tplv-k3u1fbpfcp-watermark.image)

`create-react-app`是大家常用的用来创建react项目的脚手架，它的设计理念和实现思路值的我们学习。我研究了一下`create-react-app`源码，并把它的核心功能模块梳理出来。

下面是这篇文章的主要内容:

1. 简单介绍create-react-app的使用
2. 介绍create-react-app的流程，从全局上看create-react-app是怎么创建react项目的
3. 详细的分析create-react-app的3个核心模块的实现
4. 总结

# create-react-app快速入门

1. 使用create-react-app创建项目my-app：

```
npx create-react-app my-app
```

2. 进入 my-app 文件夹，执行 `npm start` 启动项目

默认阅读这篇文章的同学都是接触过create-react-app的人，所以不对如何使用create-react-app进行深入介绍，如果想了解详细情况请阅读官网文档（[👉](https://emojipedia.org/backhand-index-pointing-right/) [create-react-app](https://www.html.cn/create-react-app/docs/documentation-intro/)）。

## 必备知识

为了更好的了解create-react-app内部的实现原理，我们需要掌握以下这几个知识点：

### 1. monorepo管理

**概念**

Monorepo 是管理项目代码的一个方案，即在一个项目仓库(repo)中管理多个模块/包(package)。Monorepo的优势在于一个仓库维护多个模块，能够统一工作流，代码共享。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4af42f39751a4f02b6205e8b8cebcace~tplv-k3u1fbpfcp-watermark.image)

create-react-app使用Monorepo方案在packages下维护了11个包。这些包互相之间有一定的联系，放在一个仓库中维护方便代码管理。此思路也值的我们在工作中学习运用。

```
.
├── babel-plugin-named-asset-import
├── babel-preset-react-app
├── confusing-browser-globals
├── cra-template
├── cra-template-typescript
├── create-react-app
├── eslint-config-react-app
├── react-app-polyfill
├── react-dev-utils
├── react-error-overlay
└── react-scripts
```

**使用**

我们可以用lerna或yarn workspace实现Monorepo方案，此处介绍lerna构建基础Monorepo仓库过程：

1. 进入项目目录，创建一个 lerna 管理的仓库

```
lerna init
```

2. 增加一个 packages

```
lerna create my-package
```

3. 发布包。提示输入新版本并更新所有在 github 和 npm的包。

```
lerna publish
```

 4. 把packages下所有包的依赖安装到根 node_modules。

```
lerna bootstrap

```

详细使用文档请查看：[👉](https://emojipedia.org/backhand-index-pointing-right/) [lerna官网](https://lerna.js.org/)

### 2. node必备模块

**commander**

[commander](https://github.com/tj/commander.js/blob/HEAD/Readme_zh-CN.md) 是一个完整的node.js命令行解决方案，封装了获取命令行指令

- `.version`方法可以设置版本，其默认选项为-V和--version
- 通过`.arguments`可以为最顶层命令指定参数，对子命令而言，参数都包括在command调用之中了。尖括表示必填（eg. ），而方括号（eg. [optional]）则代表选填。
- 通过`.usage`选项可以修改帮助信息的首行提示

如下demo表示，运行create-react-app myApp op1，其中myApp是必须要写的，op1可不写

```
const chalk = require('chalk');
const {Command} = require('commander');
new Command('create-react-app')
    .version('1.0.0')
    .arguments(' [optional]')
    .usage(`${chalk.green('')} [optional]`)
    .action((must,optional,...args) => {
       console.log(must,optional,args);
    })
    .parse(process.argv);
```

**cross-spawn**

- [cross-spawn](https://www.npmjs.com/package/cross-spawn)是node的spawn和spawnSync的跨平台解决方案

- [inherit](https://nodejs.org/dist/latest-v15.x/docs/api/child_process.html)表示将相应的stdio流传给父进程或从父进程传入

  

```
const spawn = require('cross-spawn');
const child = spawn('node', ['script.js','one','two','three'], { stdio: 'inherit' });
child.on('close',()=>{
    console.log('child is done!');
});
const result = spawn.sync('node', ['script.js','one','two','three'], { stdio: 'inherit' });
console.log(result);
```

## create-react-app各模块介绍

create-react-app的实现过程可以用下面的流程图[👇](https://emojipedia.org/backhand-index-pointing-down/)表示，最重要的是 `create-react-app`，`react-scripts`和`cra-template`这三个模块。

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b72f051e6fb0499e9f9b69f1cc31d320~tplv-k3u1fbpfcp-watermark.image)

梳理一下流程：

1. 命令行输入`npx create-react-app my-app`

2. 调用`create-react-app`模块，

   - 创建`my-app`文件夹

   - 写入`package.json`
   - 安装`react`, `react-dom`, `cra-template`, `react-scripts` 这四个模块
   - 调用react-scripts的init.js

3. 调用`react-scripts`的init.js

   - 根据`cra-template/template.json`和 `my-app/package.json`合并出新的package.json
   - 复制`cra-template/template`里的内容到my-app下
   - 安装项目依赖
   - 移除`cra-template`

4. 得到目标文件夹 my-app。其中`package.json`的scripts脚本命令调用了react-scripts模块`bin/react-scripts`文件。

下面重点介绍 `create-react-app`，`react-scripts`和`cra-template`这三个模块的具体实现。

## create-react-app核心模块实现

### create-react-app

**1. 主要功能**

`create-react-app`包是入口，用户在命令行输入`npx create-react-app my-app`会执行

- 和用户交互，获取项目名 my-app
- 创建my-app文件夹，安装`react`, `react-dom`, `cra-template`, `react-scripts` 这四个模块
- 调用`react-scripts/init.js`

**2.** **核心代码**

此处根据源码整理的逻辑图，方便大家阅读，

**3. 简化版实现**

为了便于理解，将上述逻辑简化了一下，实现了个简易版。项目地址：

可以通过npm i min-create-react-app -g 试用此模块。

### react-scripts

**1. 主要功能**

- 复制cra-template到目标文件夹
- 提供webpack的功能

**2. 实现思路**

2.1 复制cra-template到目标文件夹

2.2 提供scripts命令: 

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc90879e190947abba5c4e354257946e~tplv-k3u1fbpfcp-watermark.image)

关键代码：

1. package.json中bin字段指向./bin/react-scripts.js说明命令行中执行react-scripts xxx 命令会执行此文件。

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23221adbd1074e24853cb35013c88944~tplv-k3u1fbpfcp-watermark.image)

2. ./bin/react-scripts.js 中第27行和31行说明实际上执行的是对应的build.js、eject.js、start.js和test.js这四个文件。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7d11e5ee6c254dbbbe3726864330227e~tplv-k3u1fbpfcp-watermark.image)

3. react-scripts start命令

- 1. 设置process.env.NODE_ENV = 'development'
- 2. 获取webpack配置文件config/webpackDevServer.config.js
- 3. 调用react-dev-utils/WebpackDevServerUtils/createCompiler生成compiler
- 4. 调用/config/webpackDevServer.config.js生成serverConfig
- 5. 启动WebpackDevServer服务
- 6. 启动浏览器，打开项目页面

4. build命令

- 1. 设置process.env.NODE_ENV = 'production';
- 2. 获取webpack配置文件
- 3. 清空build目录
- 4. 拷贝public目录下的文件到build目录
- 5. 创建compiler并调用run方法进行编译

react的webpack配置文件做了很多优化配置，值的我们学习：[github.com/facebook/cr…](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/config/webpack.config.js)

### cra-template

**1. 目录结构**

```
.
├── README.md
├── package.json
├── template
│   ├── README.md
│   ├── gitignore
│   ├── public
│   │   ├── favicon.ico
│   │   ├── index.html
│   │   ├── logo192.png
│   │   ├── logo512.png
│   │   ├── manifest.json
│   │   └── robots.txt
│   └── src
│       ├── App.css
│       ├── App.js
│       ├── App.test.js
│       ├── index.css
│       ├── index.js
│       ├── logo.svg
│       ├── reportWebVitals.js
│       └── setupTests.js
└── template.json

```

**2. 主要功能**

cra-template放的是react基础项目模板，会被拷贝到目标文件夹成为基础项目文件。

- public中存放静态资源
- src中存放.js和.css文件
- template.json中有此模板依赖的package，react-scripts在复制模板到目标文件夹时会将template.json和原package.json文件合并生成新的package.json

## 总结

至此，create-react-app的核心代码已经介绍完毕。

通过这篇文章，我们了解到以下几点：

1. [create-react-app](https://github.com/facebook/create-react-app) 采用Monorepo方案，在一个仓库里管理create-react-app,react-scripts和cra-template等多个包，实现工作流和代码共享；

2. [create-react-app](https://github.com/facebook/create-react-app) 项目中，create-react-app包是入口，实现了读取命令行中的项目名，创建项目文件夹，安装react, react-dom, cra-template, react-scripts 这四个模块，最后调用react-scripts的init.js

3. react-scripts提供两块功能，一是复制cra-template到目标文件夹，二是提供webpack的功能

4. cra-template放的是react基础项目模板，会被拷贝到目标文件夹成为基础项目文件

希望这篇文章能够对你有所帮助。

## 参考

[facebook/create-react-app](https://github.com/facebook/create-react-app)

[create-react-app官网文档](https://www.html.cn/create-react-app/docs/documentation-intro/)

[基于 Lerna 管理 packages 的 Monorepo 项目最佳实践](https://www.cnblogs.com/vivotech/p/11316961.html)

[使用 MonoRepo 管理前端项目](https://blog.csdn.net/qiwoo_weekly/article/details/112000852)
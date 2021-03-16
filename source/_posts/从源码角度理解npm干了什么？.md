![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d8c3afabef14766adf570beb5a06e75~tplv-k3u1fbpfcp-watermark.image)

我在整理npm相关知识时，发现有些问题比较困惑，网上也没有从源码层面解释npm的文章，所以我去看了源码来解决我的困惑。为了加深理解，我把源码里的重点内容整理出来，希望大家在读完后也能够对npm有更深的理解。
<!-- more -->
## 什么是npm

npm全称_node package manager_，[维基百科关于Node.js](https://zh.wikipedia.org/zh-hans/Node.js#)的介绍中指明[npm](https://zh.wikipedia.org/zh-hans/Node.js#npm)是Node.js附带的包管理器。下载安装node时会附加安装了npm。npm是一个命令行工具，用于从NPM Registry中下载、安装Node.js程序，同时解决依赖问题。npm提高了开发的速度，因为它能够负责第三方Node.js程序的安装与管理。

[👉](https://emojipedia.org/backhand-index-pointing-right/) [npm官网文档](https://docs.npmjs.com/about-npm/)

[👉](https://emojipedia.org/backhand-index-pointing-right/) [npm git仓库](https://github.com/npm/cli)

## 安装

npm是Node.js附带的包管理器，也就是说安装node完后，自动安装npm。

- node可在官网下载指定版本安装 [node官网](https://nodejs.org/en/download/)

- 如果是mac电脑，也可以使用homebrew安装。打开终端，在命令行中输入

  brew install node

下载node包打开后也能看到里面默认带了npm和npx

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb1953197a2b4519a056f4049ca038ee~tplv-k3u1fbpfcp-watermark.image)

## 使用

在开发代码阶段，我们时常要使用到 npm：

- 在初始化项目时，我们会使用`npm init -y` 命令来生成`package.json`文件
- 然后再`npm install <packageName>`安装模块
- 开发代码完成后需要在package.json中配置scripts字段start，再通过`npm run start`执行代码

**那么实际上执行npm命令的调用关系是什么？实际是执行哪个脚本呢？**

我们可以在终端任意路径下输入`which npm`查看npm执行文件所在的位置，结果如下：

```
> /usr/local/bin/npm
复制代码
```

可知npm命令实际上是执行的是/usr/local/bin/npm文件。继续输入`ll /usr/local/bin/npm`

```
> lrwxr-xr-x 1 kin admin 46B 4 7 2020 /usr/local/bin/npm -> /usr/local/lib/node_modules/npm/bin/npm-cli.js
复制代码
```

也就是说_/usr/local/bin/npm_是个软链，实际执行的是_/usr/local/lib/node_modules/npm/bin/npm-cli.js_文件

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b37ae13045e04038b7e3cd98946410cf~tplv-k3u1fbpfcp-watermark.image)

## 如何调试npm源码

首先去 [npm的git仓库](https://github.com/npm/cli)把代码拉取到本地，开始调试源码。

这里介绍一个比较取巧的方式调试，先确认一下你使用的IDE是VSCode。

1. 在项目根路径的package.json中给scripts字段添加一行命令[👇](https://emojipedia.org/backhand-index-pointing-down/)

```
"debugger": "node ./bin/npm-cli.js install moment",
复制代码
```

解释一下，因为npm本质上就是`node ./bin/npm-cli.js`，故使用`node ./bin/npm-cli.js` 替代npm即可。也就是说配置的`node ./bin/npm-cli.js install moment`等价于 `npm install moment`。

2. 点击scripts字段上面的Debug小图标

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f24593bd1ab4acdbb629ea6bc480221~tplv-k3u1fbpfcp-watermark.image)

3. 弹窗中选择debugger，进入调试模式

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3eab956a25c94db88212a4531184eddb~tplv-k3u1fbpfcp-watermark.image)

## 源码解析

这里将从这几个方面分析npm源码：

1. 整体介绍
2. npm常用命令的实现，包括

- `npm init`
- `npm install <packageName>`
- `npm run scripts`

### 1. 整体介绍

根据上面的分析可知，npm实际执行的文件是/usr/local/lib/node_modules/npm/bin/npm-cli.js，查看npm-cli.js文件的代码可知：

```
#!/usr/bin/env node 
require('../lib/cli.js')(process)
复制代码
```

内部实际调用的是/npm/lib/cli.js，核心逻辑在56行，从npm.commands上获取指令所对应的文件，如果存在就执行此文件。

```
const impl = npm.commands[cmd]
if (impl)
  impl(npm.argv, errorHandler)
else {
  npm.config.set('usage', false)
  npm.argv.unshift(cmd)
  npm.commands.help(npm.argv, errorHandler)
}
复制代码
```

也就是说npm run xxx命令最后会执行相对应的文件

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20a8459002ce459598f49a95ce7c7fd8~tplv-k3u1fbpfcp-watermark.image)

### 2. npm常用命令的实现

重点介绍这几个常用命令`npm init`，`npm install <packageName>`和 `npm run <scripts>`

**npm init。**

如上分析可知，`npm init` 会对应执行[lib/init.js](https://github.com/npm/cli/blob/latest/lib/init.js)，核心代码从71行开始，调用initJson()。

```
await new Promise((res, rej) => {
    initJson(dir, initFile, npm.config, (er, data) => {
      npm.log.resume()
      npm.log.enableProgress()
      npm.log.silly('package data', data)
      if (er && er.message === 'canceled') {
        npm.log.warn('init', 'canceled')
        return res()
      }
      if (er)
        rej(er)
      else {
        npm.log.info('init', 'written successfully')
        res(data)
      }
    })
  })
复制代码
```

 initJson是本质上调用的是init-package-json 中的init方法。主要做的就是写入package.json文件

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4772ce381e9e49ba8baa182cb001101f~tplv-k3u1fbpfcp-watermark.image)

## 总结

读完这篇文章，你应该对npm的实现机制有了基本的了解：

1. npm是node.js附带的包管理器，在安装node时会附带安装上

2. npm本质是一段node脚本，我们执行npm命令其实就是执行`node /usr/local/lib/node_modules/npm/bin/npm-cli.js`

3. 最后分享一点阅读源码的心得：

- 读源码时要抓大放小，先把整体脉络整理出来，在针对各个细节进行分析；
- 抓住核心逻辑，读进去还能读出来，就是能表达出源码做了啥；
- 进阶段位就是学习源码里的优秀思想，学以致用吧

—

希望这篇文章能够给你有所帮助，对文章中的内容有疑问的地方欢迎一起探讨。
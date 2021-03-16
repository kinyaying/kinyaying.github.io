koa是经常使用的node端框架，它封装了一系列node方法，通过它的api让写服务变得更加方便。而且相比express，koa支持promise写法，更加符合现在的前端代码编写习惯，代码可读性更强。koa相比express更加短小，更多的功能是通过插件的形式实现的，它的设计思路很值得我们参考。

本篇文章将koa的逻辑用流程图表示，让我们理解起来更加简单。
<!-- more -->
# 代码结构
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae9cf32b2dd9416087eb83164269820e~tplv-k3u1fbpfcp-watermark.image)

koa的代码十分短小精悍，有效的代码文件就lib目录下的4个文件，从package.json里的`main`字段可知入口文件是`lib/application.js`。koa的主要功能由application.js提供，context.js，request.js和response.js 这三个文件主要是对原生req和res进行扩展，给koa的请求上下文ctx提供更丰富的功能。

# Koa创建服务
```js
const Koa = require('koa');
// 1. 创建koa应用
const app = new Koa();
// 2. koa中间件
app.use(async ctx => {
  ctx.body = 'Hello World';
});
// 3. 起服务
app.listen(3000);
```

如上示例代码，三步就能够起一个简单的koa服务。[👉 demo1](https://github.com/kinyaying/koa-demo/blob/main/demo1.js)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9cd91d1c7594d759ad147c7fa7468b4~tplv-k3u1fbpfcp-watermark.image)

起服务各个步骤的详细解析如下：

**1. new Koa()**
调用了koa的constructor方法，进行了context、request、response的初始化

**2. app.use()**
将中间件函数存储在middleware中

**3. app.listen()**

- 起http服务
- 监听请求，如果有请求进来，创建koa的请求上下文ctx，依次调middleware里的中间件函数对请求进行处理。


# 中间件原理
koa的中间件使用的是洋葱模型，具体我们通过下面两个例子深入了解。
### 1. koa中间件使用
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4cb22cf5816444798038a1cee5eb9aa~tplv-k3u1fbpfcp-watermark.image)

**情况一： `await next()`**

koa中间件正常的使用需要在next前加await，这样能确保请求被每一个中间件都处理完再返回响应。[👉 demo2](https://github.com/kinyaying/koa-demo/blob/main/demo2.js)
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7431b034d26541ed85f64f96b3e8833c~tplv-k3u1fbpfcp-watermark.image)

**情况二： 直接`next()` ，没用`await`**

如果`next`前没有加`await`，那么中间件1相当于同步代码，如果中间件2有异步代码，中间件1不会等待中间件2执行完而是直接返回。这样会导致请求没有被后面的中间件处理就返回了，我们在应用中尽量避免此情况。[👉 demo3](https://github.com/kinyaying/koa-demo/blob/main/demo3.js)

**小结：**

我们在使用koa中间件时确保使用的规范性，用`await next()`这种方式，而不是`next()`这种。因为我们没法确保每个中间件都没有异步代码，使用`await next()`能等待异步代码执行完毕再往下执行。

### 2. koa中间件原理
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6765b516fbb7400299151aac96dbfc96~tplv-k3u1fbpfcp-watermark.image)

koa中间件是通过[koa-compose](https://github.com/koajs/compose)实现的。为了方便理解，我将compose函数简化一下。

**compose函数简化版**
```js
function compose(middlewares) {
  // koa 核心代码
  const dispatch = (i) => {
    if (i === middlewares.length) return Promise.resolve()
    return Promise.resolve(
      middlewares[i](() => {
        dispatch(i + 1) // 当用户调用next时会取出下一个继续执行
      })
    )
  }
  return dispatch(0)
}
```

compose执行时会先通过`dispatch(0)`执行middlewares的第一个函数，传入`()=>{dispatch(1)}`作为next。中间件1运行到next时会触发`dispatch(1)`的执行。`dispatch(1)`则会执行中间件2，并且传入`()=>{dispatch(2)}`作为next。中间件2执行到next()时相当于执行`dispatch(2)`，从而触发中间件3执行...
具体的执行过程可以下载 👉 [compose精简版](https://github.com/kinyaying/koa-demo/blob/main/compose.js)，看简化版compose的运行结果。

# koa上下文(ctx)

koa中使用的环境变量是ctx。通过`Object.keys(ctx)`可知ctx上有 `request, response, app, req,  res, originalUrl, state`这几个属性。其中ctx.res, ctx.req是原生的请求和响应；ctx.response，ctx.request是针对原生请求的扩展。

koa要保证不同请求的ctx保持独立，并且将ctx的属性获取代理到response.js和request.js上，简化ctx取值操作。

**1. koa的context实现了不同应用以及不同请求之间的环境隔离**
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0018165c33749e3a9412d1ebdf3b433~tplv-k3u1fbpfcp-watermark.image)

如图所示：

① 每个koa应用都会拥有自己的context, (源码：[koa/application.js](https://github.com/koajs/koa/blob/master/lib/application.js) 59行) 实现应用的隔离。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b706081401b149a081e7d09ecc9986ef~tplv-k3u1fbpfcp-watermark.image)

② 每个请求都会通过Object.create()拥有自己的ctx, (源码：[koa/application.js](https://github.com/koajs/koa/blob/master/lib/application.js) 178行) 实现请求的隔离。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8601ffa627734426a11353ceaac7b764~tplv-k3u1fbpfcp-watermark.image)

通过两次的`Object.create` koa确保每次请求都能拥有独立的ctx变量，避免不同请求的ctx变量互相污染。

**2. ctx能直接取request或response的值，因为ctx被代理到request.js和response.js上，这给开发带来了便利。**

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cced934c28374cc4b84c436a41cb5d7d~tplv-k3u1fbpfcp-watermark.image)

我们在编码时可以直接写`ctx.path`来获取当前请求的路径，这是原生的req不提供的。当从ctx上取属性path时，因为ctx被做了代理，实际上是从`cxt.request`上取path的（如图上步骤①，对应源码 [koa/context.js](https://github.com/koajs/koa/blob/master/lib/context.js)  253行）。而`cxt.request`是request.js提供的（如图上步骤②， 对应源码  [koa/application.js](https://github.com/koajs/koa/blob/master/lib/application.js) 179行），它能通过`this.req`获取到原生req（源码 [koa/application.js](https://github.com/koajs/koa/blob/master/lib/application.js) 182行），然后经过解析处理转化成url返回给 `ctx.path`。


![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc0ca7edb85647ca93ca0f460fc27c29~tplv-k3u1fbpfcp-watermark.image)

再举一个例子，当我们写入`ctx.body='ok'`时，实际上ctx被代理到`ctx.response`上，在response.js内通过`set body()`写入this._body变量。当返回请求时，通过`get body()`获取并返回this._body。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/879aafcdeb774e48b94b27ac97322fe0~tplv-k3u1fbpfcp-watermark.image)

**小结：** koa将context上的属性代理到request.js和response.js上，通过属性劫持给context扩展了一些新的属性和方法。(源码[koa/context.js](https://github.com/koajs/koa/blob/master/lib/context.js) 199-251行)。我们可以直接从ctx上获取对应的属性和方法，给代码的编写提供了便利。

# 总结

希望读完这篇文章能够让你对koa的实现原理有进一步的了解。koa短小精简的代码非常适合阅读。这篇文章主要介绍了：
1. koa中间件实现使用了洋葱模型。
2. koa每次请求都会创建一个新的请求上下文，保证了context的请求隔离和应用隔离。
3. koa将context上的属性代理给response.js和request.js处理，扩展了context功能。


# 参考
[koa官网](https://koa.bootcss.com/)

[koa源码](https://github.com/koajs/koa)

[koa-compose源码](https://github.com/koajs/compose)

[Koa源码浅析](https://segmentfault.com/a/1190000019603834)

—

希望这篇文章能够给你有所帮助，对文章中的内容有疑问的地方欢迎一起探讨。



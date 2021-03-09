---
title: cookie、session和jwt
date: 2021-02-09 13:58:06
tags: 浏览器
keywords:
  - cookie
  - session
  - jwt的实现
---

# 概要

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ee0f3f7310542f6bace07526605f510~tplv-k3u1fbpfcp-watermark.image)

# cookie

## 1. 为什么会出现 cookie?

http 是无状态的协议，导致请求不知道是哪个用户操作的。cookie 的出现就是为了解决 http 请求无状态和服务端要知道请求来源之间的矛盾。

cookie 会根据服务端发送的响应报文中的 `set-cookie` 字段信息，通知客户端保存 cookie。下次客户端发送请求时就会携带 cookie，服务端收到请求，根据 cookie 和服务器的记录进行比对，找到之前的状态。

<!-- more -->

## 2. cookie 存在的问题有哪些？

- cookie 会作为 http 的请求报文进行传递，所以要注意 cookie 字段的大小，一般小于 4kb。如果 cookie 存储的字段过大，浪费流量，服务端也会耗费性能解析 cookie。

- cookie 保存在客户端，存在篡改或劫持的风险。故 cookie 不能存储重要信息，一般只保存凭证，服务端会根据 cookie 从数据库或 session 中查找具体信息。

关于 cookie 更详细的介绍可以看维基百科 ➡️ [HTTP cookie](https://en.wikipedia.org/wiki/HTTP_cookie)

# session

## 1. session 是什么

session 本质是一段保存在服务端内存中的代码片段，session 的实现一般是基于 cookie 的。cookie 记录凭证，session 记录具体数据。

## 2. session 存在问题

- session 会占用内存，开销大。传统的 session 保存在内存里，每当用户登录时，在 session 做一次记录。随着认证用户的增多，服务端的开销会明显增大。
- session 横向扩展差。页面的请求不一定是同一台服务器，如果请求打到不同服务器，那么服务器之间需要共享 session。此时需要做 session 的持久化，如果持久化失败就出现认证失败。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4fbf18972344e079ba04a5648b3a63b~tplv-k3u1fbpfcp-watermark.image)

# jwt

jwt 全称 json web token，是目前最流行的跨域身份验证解决方案。

## 1. 解决的问题

传统的 session 不支持分布式架构，无法支持横向扩展，只能通过数据库来保存会话数据实现共享。如果持久层失败会出现认证失败。回到问题的本质，是因为服务端需要记录客户端对应的信息，来鉴别客户端状态。
而使用 jwt，服务端不需要记录客户端对应的信息。服务端通过对 token 上携带的信息进行处理能够确认这个 token 是否是有效。服务器变为无状态，使其更容易扩展。

## 2. token 的组成

jwt 生成的 token 由 3 部分组成：头、内容、签名，通过符号`.`连接。

1. Header 头部

- 声明类型，这里是`JWT`

- 声明加密的算法，这里是 `HS256`

  Header 的示例：

  ```js
  {
  'typ': 'JWT',
  'alg': 'HS256'
  }
  ```

2. Payload 内容：存放有效信息，例如过期时间，面向用户等

   Payload 的示例：

   ```js
   {
   "exp": "1234567890",
   "name": "John Doe"
   }
   ```

3. Signature 签名

   Signature 由下面两步得到

   1. 将 base64 加密后的 Header 和 base64 加密后的 Payload 使用.连接组成的字符串

   2. 将字符串进行 Header 中声明的加密方式进行加盐 `secret` 组合加密

      ```js
      HMACSHA256(
        base64UrlEncode(header) + '.' + base64UrlEncode(payload),
        secret
      )
      ```

## 3. 鉴权原理

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64f115797dad43a7bc92a114442c2694~tplv-k3u1fbpfcp-watermark.image)

① 客户端发送/login 请求到服务端 A，服务端 A 校验登录通过后生成 token。

② 服务端 A 将 token 返回给客户端。

③ 客户端发送/list 请求到服务端 B，同时在请求头里携带了 token。服务端将 token 拆解成 header+payload+signature 三部分，再次将` header.payload` 进行加密，得到新的 signature。如果`新的 signature` 和`旧的 signature` 相等则表示当前客户端是登录过的状态。

④ 服务端 B 校验通过，返回数据给客户端。

## 4. 常用的 jwt 库有

- [jwt-simple](https://www.npmjs.com/package/jwt-simple)

- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken)

# 总结

读完这篇文章，希望你能对 cookie、session 和 jwt 有一定的了解。

传统的 cookie+session 的鉴权形式有一定的局限性，表现在：

1. session 占用服务器内存
2. session 的横向扩展性不好，需要数据持久化。数据持久化如果失败影响鉴权。

使用 jwt 能让服务器变成无状态的，即服务器不需保存当前请求的会话信息。实现原理是：

- 在初次请求时，服务端生成一个 token 返回给客户端；
- 客户端再次请求时携带 token，服务端将 token 拆解成 header+payload+signature 三部分，将 `header.payload `进行加密，得到新的 signature。如果`新的 signature` 和`旧的 signature` 相等则表示通过，当前客户端是登录过的状态。

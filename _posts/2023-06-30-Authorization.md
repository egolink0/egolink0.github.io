---
title: 用户状态记录的方式：Cookie、Session、Token
date: 2023-06-30 8:00:00 +0800
categories: [文章, auth]
tags: [Cookie, Session, Token, jwt]
author: ego
---

![op](/assets/img/blog/auth-op.png)

客户端和服务端进行的用户会话状态存储的机制变化，大致是：

- Cookie 机制(客户端存状态)
- session 机制（服务端存状态）
- Token（客户端存状态）。

## 1. cookie 机制

一般同一个用户发送的多次请求用的都是同一个身份，但是 http 不记录用户身份，所以服务端无法知道上一次和这一次的请求是同一个用户发送的。这时候就需要额外的机制来辅助存储用户的信息，就是 cookie。

一般我们所说的 cookie 包含了两层意思，一个是浏览器的 cookie 存储方式，类似 localStorage。另一个就是用来身份校验的会话状态记录机制：cookie 机制。

cookie 机制是基于 cookie 存储方式实现的状态记录。

### 浏览器 cookie 存储的特点

- 不支持跨域
- 一些手机浏览器不支持
- 有存储空间大小限制，4kb

### cookie 机制的流程

#### 首次认证

- 客户端：用户输入用户名和密码，发送给服务端
- 服务端: 校验成功，返回 uid 等信息
- 客户端：通过 javascript 写入本地 cookie （存下 username，password（加密），uid 等信息）

#### 二次发送

- 客户端：发送请求，携带 cookie 信息
- 服务端：校验成功，返回请求的结果
- 客户端：收到结果，显示内容

![auth-cookie](/assets/img/blog/auth-cookie.png)

> cookie 机制中存在的问题是：
>
> 1. 每次浏览器都要校验用户名和密码，需要查库验证，每次都要传用户名和密码
> 2. 受限于 cookie 的存储特点：不能跨域，有些手机不支持，大小有限制

## 2. session 机制

因为 cookie 机制每次都要验证的问题，诞生了基于 cookie 存储结构的 session 机制。 session，翻译过来就是会话的意思。

### session 机制流程

#### 首次认证

- 客户端：用户输入用户名和密码，发送给服务端
- 服务端：校验通过后生成 session 信息，包含 sessionId 和 sessionValue（一般包含用户的基本信息，uid,tier，expireTime 等）,设置了过期时间的 session 会在过期后自动删除。
- 服务端：返回的 http response header 中设置 `Set-Cookie:sessionId=xx` ，然后客户端收到响应，客户端的 cookie 里就多了一条 sessionId。

#### 二次发送

- 客户端：用户二次访问，配置请求的连接带上 cookie 信息，就会在 request 的 header 里面多一条 `Cookie:sessionId=xx`
- 服务端：收到 sessionId，在存的 session 信息中找 sessionId 对应的 sessionValue。
- 服务端：拿到用户 sessionValue 中的 uid ，直接就可以使用 uid 访问数据库的资源，不用二次验证 用户名和密码
- 服务端：返回资源

![auth-session](/assets/img/blog/auth-session.png)

### 什么时候需要 redis ?

> redis 是一个数据库，以 `key:value` 的形式存储数据。  
> 在上述流程中，一个服务生成的 session 信息只存储在那个服务的内存中，但是实际应用中，一般会起很多个服务来响应用户的需求，用户的请求如果下次打到另一个服务上，那个服务没有 session 信息就会需要重新登录，所以一般情况下，会将 session 存储在额外的服务中，类似数据库，多个服务都可以连接到这个库中，查询 session 的信息是否存在，完成共享，一般情况下，session 的外部存储大多用的是 redis。

> 所以可以看到 session 机制中存在的问题是：
>
> 1. 受限于 cookie 存储特点：不能跨域，有些手机不支持，大小有限制
> 2. 需要存储空间（内存或者额外的数据库，比如 redis），相应的，可能就需要额外的查库时间

## 3. Token 机制

session 机制很好的解决了 cookie 机制的第一个问题，但仍然是建立在 cookie 存储方式上的一种会话状态存储机制，cookie 存储的缺点仍然在：如不能跨域，有些手机不支持，大小有限制，甚至有 CSRF 攻击的风险。所以诞生了一种 Token 机制，其中 JWT 是普遍认同的 Token 机制中较优的一种。

下面是一个 token 机制流程，主要用的 jwt 的结构。

### token 机制流程

#### 首次验证

- 客户端：发送用户名和密码
- 服务端：验证通过，生成 jwt 格式的 token 返回给客户端，token 包含一些用户信息，比如 uid
- 客户端：存储 token，可以用 `cookie/localStorage/sessionStorage` 等存储

#### 二次发送

- 客户端：将 token 放到 http 请求的 `Authorization` 字段中，传给服务器
- 服务端：解析 token 是否正确有效，通过后根据 token 中的用户信息 uid 等进行后续资源访问，获取请求结果
- 客户端：收到结果，显示

![auth-token](/assets/img/blog/auth-token.png)

### token 优势

可以看到 token 的方式解决了 cookie 机制和 session 机制中的几个问题：

- 可以不用 cookie 的方式存储，可以存在 localStorage 等，相应的不能跨域、大小限制、安全问题，客户端不支持等问题也相应的解决了
- 服务端不用额外存储用户信息，减少了内存或者数据库（存储和查询）开销

### JWT（[JSON Web Token](https://jwt.io/introduction)）

JWT 是 Token 机制中一种较优的存储结构。大致结构是 `xxxxx.yyyyy.zzzzz` 这样的，代表了 `header.payload.signature`，具体结构介绍可以点 [这里](https://jwt.io/introduction)，一段 jwt 结构的 token，也可以在 [官网](https://jwt.io/) 解析出来。

## 参考

- [傻傻分不清之 Cookie、Session、Token、JWT](https://juejin.cn/post/6844904034181070861#heading-6)
- [Cookie 与 Session 机制](https://zhuanlan.zhihu.com/p/21275207)
- [ JWT ](https://jwt.io/)

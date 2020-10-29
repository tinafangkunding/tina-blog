---
title: Serverless Dashboard 设计解读
tags:
  - Serverless
  - Dashboard
---

## 0. 前言：并不完美的 Serverless 

作为腾讯云 Serverless 的产品经理，除了自己接触 Serverless 开发外，每天也会面对非常多的开发者，一方面会感受到开发者对于 Serverless 技术的热情，另一方面也会收集到非常多的吐槽、问题和反馈，总结起来，在使用 Serverless Framework 时，客户往往有几个“不吐不快”痛点：

- 本地、云端环境不一致导致的开发、调试的问题。例如一些依赖库安装和本地调试成功，但在云端部署则失败。这种情况下进行版本迭代、问题定位都不够友好。
- 监控、告警能力的缺失：针对 Serverless 应用，目前资源级别的监控需要去手动配置。但如果涉及到应用内部的监控，则很难直观查看，从而导致发现问题、定位问题的流程较长。
- 怎样组织 Serverless 应用，不同的函数之间的调用关系、环境划分、资源的管理及权限控制等。

针对上面这些痛点问题，Serverless 团队看在眼里，急在心上。终于，在开发小哥日以继夜的努力中，5 月 12 日，腾讯云 Serverless 团队正式发布里程碑特性，通过支持应用级别监控以及 Dashboard 资源管理，有效解决上述问题！下面我们就来一起了解一下本次发布的新特性吧~

## 1. Serverless Dashboard 新特性抢先看

### 1.1 应用管理

不知道是否有小伙伴曾经有过这样的烦恼？在部署了 Serverless 应用后，不知道在哪里查看部署完毕的资源信息、实例状态和部署记录。过段时间想要再次查询也难以追溯。那么本次发布的**应用管理页面**则以 Component 为粒度，聚合了所有 Serverless Framework 部署的资源，并且展示了实例状态、访问链接以及上次的部署信息。此外，在管理详情中还支持删除 Serverless 应用、下载项目代码进行二次开发等操作，开发者可以更方便、集中的管理账号下的 Serverless 应用。

![应用列表](https://img.serverlesscloud.cn/2020511/1589216921817-1-%E5%88%97%E8%A1%A8%E9%A1%B5.png)

### 1.2 部署详情及输出

Serverless Framework 的特性之一就是可以便捷的联动关联的云上资源，因此不同的 Serverless Component，可能会联动不同的云上资源，如网关、云函数、COS等。相信许多小伙伴在进行二次开发时，都想要了解每个 Component 具体创建了的资源信息。在本次发布的**部署详情页**中，不仅可以查看到 Serverless 实例的基本信息，还可以在输出（output）页面中查看到 Serverless Component 对应的输入、输出信息。通过该页面，可以查看到对应的资源配置，如地域信息、资源id、使用的语言环境、支持的协议信息等。有了这个页面，可以直观的看到对应的资源配置，再也不担心不同应用之间搞混配置啦。

![部署详情](https://img.serverlesscloud.cn/2020511/1589216954783-2-tab%E9%A1%B5.png)

### 1.3 应用级别监控

当前 Serverless Framework 已经支持了多种 Web 框架的一键部署。在部署完毕后，相信许多开发者会希望查看到基于应用级别的监控数据。而这往往在基础资源的监控中是难以体现出来的。那么本次发布最为亮眼的能力，即支持了**应用级别的监控页面**，实现了”0“配置的监控指标展示。当前已经支持 Express.js Component 的应用级别监控。无需去多个产品的控制台查看监控，无需自助上报数据，无需借助第三方 APM 插件，只需一次部署，立刻查看 Express 应用的监控信息！

<!--more-->

当前的 Express.js 组件监控主要支持下列指标，如下图所示：

```text
函数触发次数/错误次数：function invocations & errors
函数延迟：function latency
API 请求次数/错误次数：api requests & errors
API 请求延迟：api latency
API 5xx 错误次数：api 5xx errors
API 4xx 错误次数：api 4xx errors
API 错误次数统计：api errors
不同路径下 API 的请求方法、请求次数和平均延迟统计：api path requests
```
![Express应用级别监控](https://img.serverlesscloud.cn/2020511/1589217022846-3-%E7%9B%91%E6%8E%A7%E9%A1%B5.png)


## 2. Serverless Dashboard 设计思路

由于 Serverless Dashboard 基于新版的 Serverless Component 开发，新版 Serverless Component 主要支持以下特性：

【降低门槛】交互式的一键部署指引：对于新用户而言，只需要在终端输入 serverless 命令，即可按照引导快速部署一个 Express 或 静态网站应用。
【极速部署】将一个 Express.js 应用部署到云端只需要5-6s 的时间，使本地和云端代码可以顺畅、快速同步。
【灵活复用】支持云端注册中心，每位开发者都可以贡献自己的组件到注册中心中，便于团队进行复用。
【实时日志】支持部署阶段实时输出请求日志、错误等信息，此外支持检测本地代码变化并自动部署云端，方便的进行云端代码开发。
【云端调试】针对 Node.js 应用，支持一键开启云端 debug 能力，对云端代码打断点调试，真正实现了在云端进行开发和调试的能力，无需考虑本地环境和远端环境的不一致问题。
【状态共享】通过云端部署引擎存储应用部署状态，便于账号和团队之间共享资源，协作开发。
 
当前已支持 Tencent SCF 组件，Express.js 框架，以及 Website 静态网站托管能力，支持 PostgreSQL for Serverless的数据库能力。根据以上特性，客户可以方便的部署一个 Full Stack 全栈应用。

此外，针对 Express.js 框架的应用级别监控主要基于[腾讯云自定义监控](https://cloud.tencent.com/document/product/397)能力实现。在部署过程中，框架中使用 Serverless SDK，收集应用级别的监控信息进行自定义上报和展示。因此用户可以做到 “0”配置 查看应用级别监控指标。真正实现快速部署一个开箱即用的 Serverless 应用框架。

## 3. Serverless Express 部署实战

本次实战的 Express 应用主要用到云函数、API 网关和对象存储 COS 产品。接下来，我们将通过一个 Express.js 框架的部署实战，来体验 Serverless Framework ”0“配置，一键部署，灵活复用的特性，玩转最新发布的应用管理、监控视图等能力。

1. 点击 [Express](https://serverless.cloud.tencent.com/deploy/express/) 链接，扫码登录腾讯云账号授权，一键部署。

2. 在 [Serverless Dashboard](https://serverless.cloud.tencent.com/) 中查看部署效果，点击部署完毕后输出的页面 URL，访问并查看应用级别监控数据。当前支持 15 分钟，60 分钟，24 小时和 7 天的监控数据。过几分钟，你就可以在 Dashboard 上看到对应的监控数据拉！本次 Demo 项目的参考测试链接如下：

测试 200 请求的访问链接： https://service-xxxxx-125000000.gz.apigw.tencentcs.com/release/
测试 404 请求的访问链接：https://service-xxxxx-125000000.gz.apigw.tencentcs.com/release/404
测试 /user 路径请求：https://service-r3uhesko-1251971143.gz.apigw.tencentcs.com/release/user

3. 如果希望进行二次开发，则支持在本地安装 Serverless Framework，并点击右上角的【下载项目代码】，对代码进行修改和部署。

安装 Serverless Framework 
```
npm i -g serverless
```

更多文档资料参考：https://cloud.tencent.com/product/sls
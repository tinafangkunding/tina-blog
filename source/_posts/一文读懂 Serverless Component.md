---
title: 一文读懂 Serverless Component
tags:
  - Serverless
  - Component
---

Serverless Component 全新版本于 4 月 8 日发布，本文将从下面几个方面对本次发布的新特性进行一次解读，希望可以在使用的时候也更加了解其背后的原理和行为，目录如下：

- [<a name="1"></a>1. 什么是 Serverless Component?](#1-什么是-serverless-component)
- [<a name="2"></a>2. 以 Express 组件为例，浅析架构及部署原理](#2-以-express-组件为例浅析架构及部署原理)
  - [概念和架构说明](#概念和架构说明)
    - [1. component 参数](#1-component-参数)
    - [2. name 参数](#2-name-参数)
    - [3. org 参数](#3-org-参数)
    - [4. app 参数](#4-app-参数)
    - [5. stage 参数](#5-stage-参数)
    - [1. 注册中心 Registry](#1-注册中心-registry)
    - [2. 云端部署引擎 Deloyment Engine](#2-云端部署引擎-deloyment-engine)
    - [3. 事件中心 Event Bus](#3-事件中心-event-bus)
- [3. 执行 `serverless deploy` 之后，Component 都做了哪些事情？](#3-执行-serverless-deploy-之后component-都做了哪些事情)
- [4. 开发利器：实时日志 & 云端调试](#4-开发利器实时日志--云端调试)
- [5. Component 之间的组织和引用](#5-component-之间的组织和引用)
- [6. Serverless, 未来已来](#6-serverless-未来已来)
  - [【快速部署你的 Express.js 应用】](#快速部署你的-expressjs-应用)
- [附录： Serverless Framework 支持命令列表](#附录-serverless-framework-支持命令列表)

### <a name="1"></a>1. 什么是 Serverless Component?

Serverless Component 是 [Serverless Framework](http://serverless.com/) 重磅推出的基础设施编排能力，开发者可以方便通过 Serverless Components 构建，组合并部署你的 Serverless 应用。

这个定义依然有点抽象，我们可以通过一些实际的例子来说明他的组合方式：

云上有很多产品，这些产品都有着对应的配置。然而对于一个客户来说，我只关心我是业务实现，并不想关心底层的云资源的配置是怎样的，以及到底应该怎样组合这些产品。那么 Component 就是为了解决这个问题而出现的，它支持开发者用最少的配置，构建一个 Serverless 产品/框架/应用。

如下所示，几行 yaml 配置就可以部署一个基本的腾讯云对象存储 COS 桶：
![cos](https://img.serverlesscloud.cn/2020414/1586883964642-cos-component.png)

再进一步，如果你希望搭建一个 Express.js 的 Web 框架，那么在这过程中，如果要用云上的 Serverless 产品实现，则需要组合不同的基础产品，并且对框架进行适配和改写。相信开发者都希望尽可能避免这种改造适配的工作，而 express 组件就可以将原有应用平滑的部署到云端，无需任何改造和配置。
![express](https://img.serverlesscloud.cn/2020414/1586884007566-express.png)

到了这里，可能对于很多开发者来说，支持框架适配已经可以完成很多业务的部署和迁移了。但是对于很多客户而言，他们所需要实现的比这还要具体，例如直接希望实现一个从前端、到web服务再到数据库的网站，或是一个动态博客系统，论坛，登录功能等具体的能力。那么此时，Serverless Component 依然能够通过组合进行实现。但是这次会直接通过模板来提供这些应用的支持。

![fullstack](https://img.serverlesscloud.cn/2020414/1586884042109-full-stack.png)

相信上述概念完成后，你已经对于 Component 有个大致的了解了。那么，再深入一点，Component 这么神奇，那他在背后，究竟帮我做了哪些事呢？

<!--more-->

### <a name="2"></a>2. 以 Express 组件为例，浅析架构及部署原理

要了解到架构和原理，先要从yaml配置入手。那我们先来看一下 Serverless Express Component 的 yaml 配置，再一一进行分析：
```yaml
component: express # (required) name of the component. In that case, it's express.
name: express-api # (required) name of your express component instance.
org: test # (optional) serverless dashboard org. default is the first org you created during signup.
app: expressApp # (optional) serverless dashboard app. default is the same as the name property.
stage: dev # (optional) serverless dashboard stage. default is dev.

inputs:
  src: ./src # (optional) path to the source folder. default is a hello world app.
  region: ap-guangzhou
  runtime: Nodejs10.15
  exclude:
    - .env
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
```

#### 概念和架构说明

看到上面的 yaml 配置，相信大家对于前面的几个概念会感到困惑，那么我们先来看看这几个概念都是做什么用的：

##### 1. component 参数
component 参数用来指定注册中心中的组件名称，如果希望默认使用最新版本的 component，则可以直接采用配置 `component: express`，这样每次都会默认拉取最新版的组件；但如果希望固定版本号（在生产环境中尤其推荐），则可以指定版本号进行配置，例如 `component: express@0.0.2`。该参数是必填的。
##### 2. name 参数
注册中心中存在 express 等各种组件，但是被客户使用时，其实只是复用这个组件并创造出了一个应用实例，而用户基于这个实例进行迭代和开发。因此这里的 name 代表的就是这个 express 组件在该项目中的实例名，你可以任意命名。
##### 3. org 参数
org 即 organization。也就是组织名称。用于多人协作项目时，指定一个团队所属的组织名称。
##### 4. app 参数
app 即具体的 Serverless 应用的名称，用于为资源等划分相同的项目。app 的默认值和 name 相同
##### 5. stage 参数
stage 即阶段和环境的划分。用户通过定义不同的阶段，例如 dev, test, prod 等，实现资源和环境的安全隔离，避免开发和测试影响到生产环境。同时，Serverless Component 也支持通过不同的 stage 指定不同的环境变量进行区分和管理，这块后面会进一步说明。

那么除了概念之外，了解底层架构有助于我们更好的理解其工作原理。那么在 Serverless Framework 框架中，主要由以下三个部分组成：

##### 1. 注册中心 Registry

注册中心主要为 Serverless Components 提供了一个完整的 发布、获取和列表展示的平台，同时提供 Component 的版本管理能力。每位用户支持在注册中心发布自己的 Component，也可以从云端列举，查询和下载自己所需的 Component。

##### 2. 云端部署引擎 Deloyment Engine

云端部署引擎用于存储注册中心中的 Component 及不同的版本，在客户指定某个 Component 时，部署引擎会创建一个对应的实例。此外，云端部署引擎还会在云端存储部署状态，便于团队之间项目的状态共享和协作。

##### 3. 事件中心 Event Bus

事件中心主要用于接收和转发 Serverless 平台的所有事件信息，用于承载平台内部和外部的所有通信。对于实时日志的传输，也会通过实践中心的 Websocket API 来通信。

### 3. 执行 `serverless deploy` 之后，Component 都做了哪些事情？

以一个 Express.js 应用为例，在你执行了 `serverless deploy` 后，对应上述 yaml 配置，Serverless Framework 主要做了如下几件事：

1. 根据 yaml 中的 component 名称，在云端注册中心中寻找对应的 Component 并加载。
2. 唤起扫码登录，支持用户通过微信扫码登录腾讯云，并且操作对关联资源进行授权。获取临时秘钥，绑定对应的 CAM 角色。
3. 将 Express.js 的业务代码打包压缩（yaml 对应的 `inputs` 中 `src` 目录中的代码），并且上传到特定命名规范的 COS bucket (bucket 命名为 `sls-cloudfunction-{$region}-code-{$appid}`)
4. 创建云函数，并上传 COS bucket 中的代码到云函数
5. 创建 API 网关，将 API 网关的后端指定为步骤 4 创建的云函数
6. 部署完毕，输出对应的资源信息，应用状态和访问链接等。

### 4. 开发利器：实时日志 & 云端调试

新版本的 Serverless 支持了一个重磅命令： `serverless dev`，支持开启**开发模式**，动态检测本地代码和配置的变化，并且快速部署更新，输出实时日志。对于 Node.js 10 的环境，还可以支持云端的调试能力，拿我的 Express.js 为例，可以基于 Chrome 的 Devtools 直接给云端的代码断点调试。详情可见[云端调试](https://cloud.tencent.com/document/product/1154/43220)

![云端调试](https://img.serverlesscloud.cn/2020415/1586923679446-logs.jpg)

### 5. Component 之间的组织和引用

细心的小伙伴可能已经发现，新版本 Component 和老版本有一个很大的区别，就是在新版本的 `serverless.yaml` 中，目前只能写一个 Component，而之前可以写很多 Component。那么对应的，一个 Serverless 应用之间的组织方式也会发生变化，这里可以简单介绍一个组件之间组织和引用的方案：

现在一个 full-stack 项目的目录结构：

```
.
├── frontend
|   └── serverless.yml # website 组件
└── api
    └── serverless.yml # express 组件
```

这样做的好处在于，一个 full stack 的应用可以分别部署对应的前端组件和web组件等，不需要每次的小改动都重新部署一遍项目，从而实现了业务模块之间的解耦。那么，在这样的组织结构下，组件之间的互相引用则可以通过 `${output:[app]:[stage]:[instance name].[output]}` 的方式实现：

```yaml
org: yugasun
app: fullstack-serverless-db
stage: dev
component: website
name: fullstack-frontend-v2

inputs:
  region: ${env:REGION}
  bucketName: fullstack-serverless-db
  src:
    src: ./
    hook: npm run build
    index: index.html
  env:
    # get api url after below api service deployed.
    apiUrl: ${output:${stage}:${app}:fullstack-api-v2.apigw.url}
```
如上述配置所示，在 website 静态页面组件中，环境变量配置中通过`${output:${stage}:${app}:fullstack-api-v2.apigw.url}` 引用了 express 框架组件的 API 输出。从而实现了前端页面和 API 端的传输。此外需要注意的是，只有相同 org 下的组件才能相互引用，因此在 output 中也指定了特定的 app 和 stage 来确定具体的实例输出。更多引用相关的能力，可以参考[Component 官方说明](https://github.com/serverless/components/blob/master/README.cn.md#%E5%8F%98%E9%87%8F)

参考项目：[部署 Full stack Serverless 应用](https://github.com/yugasun/tencent-serverless-demo/tree/master/fullstack-serverless-db-v2)

### 6. Serverless, 未来已来

看了这么多理论，什么都不如实践一下了解的更深入。那么通过下面的链接，快速尝试部署一个 Express.js 应用吧！

#### [【快速部署你的 Express.js 应用】](https://serverless.cloud.tencent.com/deploy/express)

### 附录： Serverless Framework 支持命令列表
```
serverless registry
```
查看可用的 Components 列表
```
serverless registry publish
```
发布 Component 到 Serverless 注册中心

--dev - 支持 dev 参数用于发布 @dev 版本的 Component，用于开发或测试。
```
serverless deploy
```
部署一个 Component 实例到云端

--debug - 列出组件部署过程中 console.log() 输出的部署操作和状态等日志信息。
```
serverless remove
```
从云端移除一个 Component 实例

--debug - 列出组件移除过程中 console.log() 输出的移除操作和状态等日志信息。
```
serverless info
```
获取并展示一个 Component 实例的相关信息

--debug - 列出更多 state.
```
serverless dev
```
启动 DEV MODE 开发者模式，通过检测 Component 的状态变化，自动部署变更信息。同时支持在命令行中实时输出运行日志，调用信息和错误等。此外，支持对 Node.js 应用进行云端调试。
```
serverless login
```
支持通过 login 命令，通过微信扫描二维码的方式，登录腾讯云账号并授权对关联资源进行操作。
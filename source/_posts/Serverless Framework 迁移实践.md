---
title: Serverless Framework 迁移实践
tags:
  - Serverless
---

## Serverless Framework 迁移实践

本文以一个 Express.js 应用为例，分享如何将 Component 切换到最新版本，使用最新的开发调试模式等。

### 新老版本 Component 有什么区别？

新版本 Component 支持以下特性：

1.【降低门槛】交互式的一键部署指引：对于新用户而言，只需要在终端输入 serverless 命令，即可按照引导快速部署一个 Express 或 静态网站应用。
2.【极速部署】将一个 Express.js 应用部署到云端只需要5-6s 的时间，使本地和云端代码可以顺畅、快速同步。
3.【灵活复用】支持云端注册中心，每位开发者都可以贡献自己的组件到注册中心中，便于团队进行复用。
4.【实时日志】支持部署阶段实时输出请求日志、错误等信息，此外支持检测本地代码变化并自动部署云端，方便的进行云端代码开发。
5.【云端调试】针对 Node.js 应用，支持一键开启云端 debug 能力，对云端代码打断点调试，真正实现了在云端进行开发和调试的能力，无需考虑本地环境和远端环境的不一致问题。
6.【状态共享】通过云端部署引擎存储应用部署状态，便于账号和团队之间共享资源，协作开发。

而老版本的 Component 的原理如下：

1. 需从 npm 安装 Component 到本地
2. 本地存储部署状态
3. 无法支持实时日志、云端调试等能力

### 怎样区分新老版本？兼容性如何？

Serverless Framework 1.67.2 版本及以上支持新版本 Component，以如下 express 应用为例，新版的 yaml 有如下特点：

```json
component: express # (required) name of the component. In that case, it's expre$
name: express-api # (required) name of your express component instance.
org: test # (optional) serverless dashboard org. default is the first org you c$
app: expressApp # (optional) serverless dashboard app. default is the same as t$
stage: dev # (optional) serverless dashboard stage. default is dev.

inputs:
  src: ./src # (optional) path to the source folder. default is a hello world a$
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

1. component 的引用方式改变了，同时新增了 `name`, `org`,`app`和`stage`字段
2. 在 `inputs` 中，增加了 `src` 配置，代表的是源码的路径

<!--more-->

### 迁移过程

#### 1. 复制项目文件

```
$ cp -R ./express-proj ./express-proj-new
```

#### 2. 修改 yaml 文件为新版本

进入新的项目文件夹
```
$ cd -R express-proj-new
```

打开 serverless.yml，修改为如下格式：
```json
component: express # (required) name of the component. In that case, it's expre$
name: express-api # (required) name of your express component instance.
org: test # (optional) serverless dashboard org. default is the first org you c$
app: expressApp # (optional) serverless dashboard app. default is the same as t$
stage: dev # (optional) serverless dashboard stage. default is dev.

inputs:
  src: ./ # 确保存放了 express 代码的目录和 src 配置的路径一致
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


#### 3. 重新部署

由于不同版本所需申请的权限不同，需要删除本地的.env_temp 文件，并且确保项目代码和上一步骤中yaml的配置路径一致，之后再次部署

移除之前的临时秘钥信息：
```
$ rm .env_temp 
```

再次部署，可以看到已经识别新版 component 
```
$ sls --debug 

Initializing...
Action: "deploy" - Stage: "dev" - App: "expressApp" - Instance: "express-api"
Deploying...

region: ap-beijing
apigw: 
  serviceId:   service-f2iva2cv
  subDomain:   service-f2iva2cv-1251971143.bj.apigw.tencentcs.com
  environment: release
  url:         https://service-f2iva2cv-1251971143.bj.apigw.tencentcs.com/release/
scf: 
  functionName: express_component_zt2i58
  runtime:      Nodejs10.15
  namespace:    default


12s › express-api › Success
```

访问新版本部署成功的链接，如果可以正常返回请求则迁移成功。

#### 4. 移除旧项目

迁移完毕后，回到旧的项目文件，将对应的资源移除：

```
$ cd ../express-proj
$ sls remove --debug

  DEBUG ─ Flushing template state and removing all components.
  DEBUG ─ Removed function express_component_7a39es successful
  DEBUG ─ Removing any previously deployed API. api-4jjcpix2
  DEBUG ─ Removing any previously deployed service. service-0bzd0fzf

  11s › express › done
```

注：该方式迁移会重新部署 Express.js 应用所对应的网关、函数等资源，因此如果绑定了自定义域名等，需要重新进行配置。
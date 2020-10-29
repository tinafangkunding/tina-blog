---
title: 不改一行代码！无损迁移 Serverless SSR 博客项目到云端
tags:
  - Serverless
  - SSR
  - Node.js
authors: 
  - tinafang
authorslink: 
  - https://github.com/tinafangkunding/
---

周五看到了 yugasun 发布的博文[如何优雅的部署一个 Serverless Next.js 应用](https://juejin.im/post/5f083245e51d4534c93974a3)，看到案例是一个基于 cnode 的论坛，于是很好奇改造一些已有的 Next.js 项目到 serverless，迁移成本如何。话不多说，直接动手实践下：

## Part 1 : 迁移流程
### 1.1 本地部署 Next.js 项目
先去 Github 上找了一些多星的 Next.js 项目，发现了一个有趣的[仿 V2EX 站点](https://github.com/sedgwickz/new_v2ex)遂 fork 了一份，下载到了本地。

为了保持包的精简，在本地安装了对应的 production 环境依赖：
```
npm install --production
```
之后通过运行 `next dev` 命令本地部署
```
npm run dev
```
根据输出，访问 `http://localhost:3000` 即可查看到效果了。so far, so easy!

### 1.2 部署 Next.js 项目到云端

根据 Next.js 组件中的案例配置文档，在项目的一级目录下分别创建下 `serverless.yml` 和 `.env` 两个文件。
```
# .env
TENCENT_SECRET_ID=xxxxxx
TENCENT_SECRET_KEY=xxxxxx
```
> `.env` 文件主要用于授权创建对应的云资源，如果项目后续要持续更新，建议直接配置永久秘钥，如不创建也没关系，可以直接通过**微信扫码**的临时授权方式来部署。 
```yaml
# serverless.yml
org: orgDemo
app: appDemo
stage: dev
component: nextjs
name: nextjsV2EX

inputs:
  src:
    dist: ./
    hook: npm run build
    exclude:
      - .env
  region: ap-shanghai
  runtime: Nodejs10.15
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
```
> `serverless.yml`详细[配置参考](https://github.com/serverless-components/tencent-nextjs/blob/master/docs/configure.md) 

之后的步骤很简单，安装 serverless framework 工具后，直接通过一行命令进行部署：

安装（ Mac 如没有权限，可以尝试下 sudo npm install -g serverless）
```
npm install -g serverless
```
部署
```
sls deploy
```
接下来，神奇的事情发生了，部署完成后（虽然等待了几分钟）直接访问输出的 URL，我惊讶的发现这个仿 V2EX 站点已经完全迁移到了云端。

![Serverless SSR](https://img.serverlesscloud.cn/2020711/1594486679493-slsdeployssr.png)

至此，一行业务代码和配置都没有改，这种体验简直不要太好。

<!--more-->

## Part 2 : 高级能力改造

体验了下刚才的站点，发现两个问题：
1. 部署时间比较久，猜测是大量的静态资源直接上传到了云函数上耗时较多；
2. 部署完毕后，首次访问加载时间较久，猜测和 1 类似，代码包大，加载的时间久，容易首次访问时触发冷启动。

接下来，打算根据 yuga 文章里的一些高级能力，开始进行优化：

### 2.1 COS 静态资源配置
首先计划把静态资源传到 COS 桶中，如果已有桶可以指定名称；如果没有的话，可以先在 `serverless.yml` 中配置并创建一个新桶：（完美避开了控制台点点点的操作）
```yaml
  staticConf:
    cosConf:
      # 这里是创建的 COS 桶名称
      bucket: serverless-nextjs
```
可以发现，再次执行 `sls deploy` 部署后，输出中多了 COS 桶的配置，嘻嘻，接下来就可以把改配置应用到 Next.js 项目中去：

COS 相关输出信息示例：
```
staticConf: 
  cos: 
    region:    ap-shanghai
    cosOrigin: serverless-nextjs-125000000.cos.ap-shanghai.myqcloud.com
    bucket:    serverless-nextjs-125000000
```
获取，`cosOrigin` 这个参数的值，修改 `next.config.js` 的配置，用于对`.next/static`目录下引用的资源进行托管

```js
// next.config.js
const isProd = process.env.NODE_ENV === "production";
const STATIC_URL =
  "https://serverless-nextjs-xxx.cos.ap-guangzhou.myqcloud.com";
module.exports = {
  env: {
    // 3000 为本地开发时的端口，这里是为了本地开发时，也可以正常运行
    STATIC_URL: isProd ? STATIC_URL : "http://localhost:3000",
  },
  poweredByHeader: false,
  assetPrefix: isProd ? STATIC_URL : "",
};
```
之后，在项目中修改引入 `public` 中静态资源的路径，比如：
```html
<!-- before -->
<head>
  <title>Create Next App</title>
  <link rel="icon" href="/favicon.ico" />
</head>
<!-- after -->
<head>
  <title>Create Next App</title>
  <link rel="icon" href={`${process.env.STATIC_URL}/favicon.ico`} />
</head>
```
改造完毕后，重新部署并访问，发现静态资源都是通过 COS 桶获取的

![COS 托管](https://img.serverlesscloud.cn/2020711/1594490574780-Screen%20Shot%202020-07-12%20at%202.02.43%20AM.png)

可以看到，加载的访问速度也有明显上升，那么是否还可以进一步优化呢？

### 2.2 配置自定义域名

为了让站点更容易被记住，配置一个自定义域名很有必要。恰好手头有一个备案的域名，因此试一下：

> 为了保证资源的安全，我为站点域名和静态资源的域名分别申请了免费的 SSL 证书，对于腾讯云上备案的域名，申请后大概 30 分钟就会审核通过，并且会帮忙自动验证。[传送门直达](https://buy.cloud.tencent.com/ssl)。

```yml
  apigatewayConf:
    protocols:
      - http
      - https
    environment: release
    enableCORS: true
    # 增加自定义域名相关配置
    customDomains:
      - domain: v2ex.tinafang.com
        certificateId: xxxxx # 证书 ID
        # 将 API 网关的 release 环境映射到根路径
        pathMappingSet:
          - path: /
            environment: release
        protocols:
          - https  
```
在部署之前，先在 [DNS 产品](https://console.cloud.tencent.com/cns)中配置了对应的 cname 解析记录。之后执行 `sls deploy` ，部署完毕后，查看到如下输出：

```
apigw: 
  customDomains: 
    - 
      created:   true
      subDomain: v2ex.tinafang.com
      cname:     service-xxxxxxx-125000000.sh.apigw.tencentcs.com
      url:       https://v2ex.tinafang.com
```

访问 https://v2ex.tinafang.com 域名，发现 v2ex 站点已经可以成功访问。

### 2.3 开启 CDN 加速

配置自定义域名后，还可以方便的开启 CDN 加速。只需要在 `serverless.yml`中加上以下配置：

```yml
    cdnConf:
      domain: static.v2ex.tinafang.com
      https:
        certId: xxxxxx
```
此外，将之前的 `next.config.js` 中静态资源获取配置修改为 CDN 地址：

```js
const STATIC_URL =
  "https://static.v2ex.tinafang.com";
```

执行 `sls deploy` 部署后，发现如下输出，在对应的域名下增加一条解析记录，把输出的 cname 记录配置上，过十分钟即可生效。

```
  cdn: 
    domain: static.v2ex.tinafang.com
    url:    https://static.v2ex.tinafang.com
    cname:  static.v2ex.tinafang.com.cdn.dnsv1.com
```

打开浏览器，查看对应的静态资源 URL，发现已经在 CDN 域名下进行加速了。

![CDN](https://img.serverlesscloud.cn/2020712/1594545364414-Screen%20Shot%202020-07-12%20at%205.15.55%20PM.png)

至此，针对站点的自定义域名、云端加速配置就全部完成了。

## Part 3 : 改造点分析

可以看到，有了 Next.js 组件，针对一个纯 Next.js 的项目确实不需要改动任何代码就可以迁移，对于前端开发者来说体验无比顺滑。

但涉及到数据库、日志、配置管理等方面时，应该免不了要针对云端做改造。接下来会梳理下这方面的开销（待续）

## 结论

走了一遍流程后可以发现，对于新手来说，迁移一个 Next.js 项目并不复杂。对于高级特性和优化，也可以在稍微修改配置和 yaml 后完成。那么，部署 Next.js 在 Serverless 架构上还有哪些好处呢？在每次执行 `sls deploy` 的时候，我发现有这样一行输出：

```
Full details: https://serverless.cloud.tencent.com/instances/appDemo%3Adev%3AnextjsV2EX
```

打开后，发现原来这种方式部署的 Serverless Next.js 应用，竟然已经提供了默认的应用管理 + 应用级别监控的能力，可以方便的看到 API 和函数级别的监控视图。

![监控视图/应用管理](https://img.serverlesscloud.cn/2020712/1594543647996-serverless.cloud.tencent.com_instances_appDemo%253Adev%253AnextjsV2EX.png)

此外，针对 Node.js 应用，在调试过程中还可以通过 `sls dev` 命令来方便的进行云端调试，输出实时日志。可以实现对项目的二次开发和快速部署。

未来，针对 SSR 应用的部署、优化，还有更多功能可以进一步探索，敬请期待！
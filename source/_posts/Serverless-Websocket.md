---
title: Serverless Websocket 实现消息实时推送--以外卖点单系统为例
tags:
  - Serverless
  - Websocket
  - Python
authors: 
  - tinafang/leonardjin
authorslink: 
  - https://github.com/tinafangkunding/
---

最近在重新整理之前 Webinar 中的例子，其中很典型的一个是 Serverless Websocket 的实现和应用。特地整理一下原理和部署过程，并且通过案例的方式分享出来。

## 1. 消息实时推送

消息实时推送的技术在很多领域都有广泛的应用。例如实时报价、在线教育、社交订阅、线上游戏等。在这些场景下，需要后台将发生的变化实时，主动地推送给浏览器端，无需用户手动刷新页面。

和移动端的 socket 方式不同，在 web 端实现消息实时推送有几种常见的方案，简要介绍如下：

#### 1.1 短轮询（polling）
轮询指的客户端和服务器端一直进行连接，每隔一段时间就请求一次。其优点是实现简单，对 HTTP 实现无需做过多修改，但缺点也非常明显，即轮询的间隔过长时客户端无法及时收到更新；此外轮询间隔短时又会导致连接数过多，增加服务器端负担。

#### 1.2 长轮询（Long-polling）
长轮询基于短轮询做了改进，即客户端发送 HTTP 给服务端时，如果没有新消息时就会等待，如有新消息时再返回给客户端。从而缓解了服务端的连接压力，并且可以保证比较好的时效性。但缺点也依然存在，因为持续的保持连接会消耗网络带宽，并且当服务端没有数据返回时，会造成请求超时。

#### 1.3 WebSocket

上述两种双向通信的实现方式并不够理想，在此背景下， WebSocket 协议出现了。WebSocket 将 TCP 的 Socket 应用在 webpage上，从而在服务端和客户端中建立了全双工（full-duplex）通信。WebSocket 连接一旦建立，无论客户端或服务端，都可以直接向对方发送报文。

相比于传统的 HTTP 每次请求和应答都需要服务端和客户端建立连接的通信方式，Websocket 在连接建立后，在断开连接前，无需服务端或者客户端重新发起连接请求。因此在并发量大的场景下，可以有效节约带宽资源，并且有显著的性能优势。

基于上述对比可知，WebSocket 协议十分适合实现实时通讯场景，一方面解决了 HTTP 协议中只能服务端发起通讯的被动性，一方面也解决了数据推送延迟的问题，并且具有明显的性能优势。

## 2. 基于 Serverless 实现 WebSocket 协议

以腾讯云为例，基于 Serverless 实现 WebSocket 协议可以参考[WebSocket 原理介绍](https://cloud.tencent.com/document/product/583/32553)。


对应到具体实现上，Websocket 在网关+函数中的实现，可以参考下图中的架构。一个 WebSocket API 会对应三个函数：

- **注册函数**: 在客户端发起和 API 网关之间建立 WebSocket 连接时触发该函数，通知云函数 WebSocket 连接的 secConnectionID。通常会在该函数中记录 secConnectionID 到持久存储中，用于后续数据的反向推送。
- **清理函数**：在客户端主动发起 WebSocket 连接中断请求时触发该函数，通知 云函数准备断开连接的 secConnectionID。通常会在该函数清理持久存储中记录的该 secConnectionID。
- **传输函数**：在客户端通过 WebSocket 连接发送数据时触发该函数，告知云函数连接的 secConnectionID 以及发送的数据。通常会在该函数处理业务数据。例如，是否将数据推送给持久存储中的其他 secConnectionID。

![WebSocket 架构](https://main.qcloudimg.com/raw/bad7c20fbf06f7bc3a1969a38795db47.png)

关于用 Websocket 实现匿名聊天室的 Node.js 版本，也可以参考下面的 [Github Repo](https://github.com/jason-alouda/websocket-nodejs)。

## 3. 部署 Serverless WebSocket 的外卖订单系统

### 3.1 订单系统演示
基于上述理论，我们可以在 Serverless 架构上部署一个 WebSocket 的点单系统。这里也有一个线上 demo 供大家参考。

- [点单系统](http://websocket-1251971143.cos-website.ap-shanghai.myqcloud.com/order.html)
- [店铺系统](http://websocket-1251971143.cos-website.ap-shanghai.myqcloud.com/shop.html)

分别打开点单系统和店铺系统，在店铺系统中点击营业，之后在点单系统中选取对应的店铺，进行下单。如下述动图所示。

![动图](https://websocket-1251971143.cos.ap-shanghai.myqcloud.com/wss.gif)

<!--more-->

### 3.2 部署流程

因为在控制台配置和部署比较麻烦，所以直接选择用 Serverless Framework 进行部署。

### 1. 安装和初始化

通过 npm 安装 serverless cli
```
npm install -g serverless
```
直接通过 Serverless registry 获取对应的模板

```
serverless init websocket-order
```

### 2. 查看目录

查看对应目录，可以看到项目中包含数据库、网关、云函数等多个子模块，说明如下：
- db 目录用于创建 PG Serverless 数据库实例
- apigateway 用于创建对应的 API ：
  - /bill 外卖下单 API
  - /get_shop_info，主要用于实现获取店铺菜单的 API
  - /pgws，用于做消息推送的 websocket API

函数列表如下：

- 消息推送相关函数：
  - 注册函数 ws_register.py
  - 传输函数 ws_trans.py
  - 注销函数 ws_unregister.py 
- 下单函数 bill.py
- 拉取店铺信息函数 get_shop_info.py
- 初始化DB函数 init_db.py 


### 3. 部署

执行 `serverless deploy` 进行项目的部署，查看对应的输出信息。

### 4. 其余配置

a. 在控制台或者 vscode 插件中，点击测试 init_db-dev 函数，对数据库进行初始化的建表等操作

b. 查看输出信息，在 function_bill 目录和 function_ws_trans 目录的 serverless.yml 中，分别配置 websocket API 的 apiid ，并重新部署两个函数，刷新环境变量配置。
```
sls deploy --target=./function_ws_trans 
sls deploy --target=./function_bill
```
c. 本地更改客户端与厨房订单系统的地址并测试
`App点单系统.html` 中更改第29行以及第88行中的 url 为生成的API网关服务域名
`店家厨房系统.html`  更改第17行 url 为API网关服务域名

url 示例：ws://service-xxxxx-xxxx.sh.apigw.tencentcs.com:80/pgws 




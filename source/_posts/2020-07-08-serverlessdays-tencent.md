---
title: 企业级 Serverless 应用实战
tags:
  - Serverless
  - ServerlessDays
thumbnail: https://img.serverlesscloud.cn/202078/1594192713077-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png
categories: 
  - news
authors: 
  - tinafang
authorslink: 
  - https://github.com/tinafangkunding/
---

本文整理自 ServerlessDay · China 大会 - 《企业级 Serverless 应用实战》的分享，讲师为腾讯云 Serverless 高级产品经理tinafang。

本文主要分为四个部分：
- Serverless 2020 : 趋势与挑战
- Serverless 典型场景
- 部署企业级 Serverless 应用
- 实战演示 : Serverless SSR

# Serverless 2020 : 趋势与挑战

首先，谈一下对于 Serverless 在 2020 的趋势。我大概是从 3-4年前开始接触 Serverless，到了今年，发现有以下一些特征，我会把他们分成三个部分：
- 第一点，对于开发者来说，Serverless 通过按需付费、弹性扩缩容的特性，极大的赋能开发者，让他们关注于实现业务，而不需要考虑底层资源。
- 第二点，对于越来越多的企业来说，从2019年开始，他们逐步开始尝试、深入使用甚至拥抱 Serverless。因为 Serverless 能够显著的降低成本，并且减少运维的工作。这对于企业来说，尤其是非科技企业来说，是有非常强的吸引力的。并且在 2020年，已经可以看出更多的企业在借助 Serverless 来实现业务了。
- 第三个方面，可以看到云服务和 Serverless 的结合越来越紧密。刚才也说到 BaaS 本身是 Serverless 中的重要部分。那么在 2020 年，越来越多的云服务，正在通过 Serverless 的方式提供。比如 PG SQL 提供了 Serverless DB ，Serverless HTTP，以及上午提到的 Serverless AI 等服务。

![Serverless](https://img.serverlesscloud.cn/202078/1594193406089-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1594193397639.png)

但是，与此同时，我们也发现，随着这些趋势的发展，也面临了不少的挑战，依然分成三个方面来讨论：

- 对于开发者来说，怎么提供一个完整的开发、调试和排障的能力，并且提供更强大的扩展能力，是非常关键的。也就是生态的建设。
- 对于企业来说，面临的问题更加细节，很多概念在工业化的实践中，都会遇到很多实际的问题。包括权限的划分、资源的管理、还有 CI/CD 等解决方案，怎样无缝适配到企业的架构中呢？
- 最后，对于云来说，结合越发紧密，但是云产品为了保证通用性和普适性，本身会有比较复杂的配置，并且云资源直接的组合需要带来比较大的学习成本，也对于企业带来了不少挑战。

![Serverless](https://img.serverlesscloud.cn/202078/1594193406087-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_1594193397639.png)

<!--more-->

# Serverless 典型场景

那么，在和企业的实践中，我们也发现 Serverless 对于几种典型的场景支持的非常优秀，在这里也希望和大家分享：
- **第一种就是 Serverless SSR**，这是一个偏前端的场景。产生背景是因为 CSR 是客户端渲染，需要浏览器端进行 js 文件的执行得到交互页面。但是缺点是 SEO 不够友善，并且首屏打开的性能不足。但是 SSR 因为涉及服务端，需要考虑 node server 的扩缩容、运维等等，让很多开发者望而却步。但是 Serverless SSR 可以很好地支持这一场景。

![Serverless](https://img.serverlesscloud.cn/202078/1594192711713-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)

- **第二种是结合网关+函数，提供通用的 RESTful 平台**，这种场景在之前 19 年一个 Serverless 的调研中，是 70% 用户都在使用的典型场景。也就是将前后端资源 Serverless 化，提供增删改查的能力。

![Serverless](https://img.serverlesscloud.cn/202078/1594192712479-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)

- **最后一个，Serverless 的全栈应用也非常有市场**。通过提供前端、后端以及数据库端。组合不同的组件，可以非常完美的支持全栈应用的部署，同时也不会失去灵活性，可以很好地拆分前后端。

![Serverless](https://img.serverlesscloud.cn/202078/1594192712657-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)

# 部署企业级 Serverless 应用

接下来谈谈部署企业级应用的几个诉求。这里的一些经验都是在实际的上云过程中，客户反馈，并且提到的非常多的问题。也是我们在帮客户一起查看问题的时候，实际解决的问题。

比如我们的一个客户，希望 All in Serverless，这几个问题他们全都遇到过，那么我们可以一起来看下是怎样解决的。

在我们帮助企业客户部署 Serverless 应用的时候，需要考虑到的几个特性：
- 权限管理
- 资源、环境的划分
- 运维、排障能力
- CI/CD

接下来，我们逐个看一下，企业客户在上云过程中是怎么解决这些问题的：

## 权限管理

当前在大企业中，需要使用主账号+子账户的用户、用户组划分权限。但是怎样让子账户之间权限隔离，更加安全的部署资源一直都是一个挑战。为了确保子账户之间的隔离和细粒度控制，Serverless Framework 开发平台支持在 serverless.yml 文件中，通过指定配置角色来获取对应权限。同时，支持运维配置不同的角色只能被某个子账户调用，从而保证其严格隔离。

![serverless](https://img.serverlesscloud.cn/202078/1594192712031-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)

## 资源&环境划分

随着企业客户 Serverless 架构上云过程中，资源的增长，不可避免会出现资源管理困难，需要有效划分资源，隔离环境的问题。那么，腾讯云 Serverless Framework 是怎样解决这个问题的呢？

主要是通过 yaml 配置中对 stage、 app 和 org 等几个字段的灵活引用，并且在控制台中提供开箱即用的资源管理视图的查看，从而有效的隔离不同环境中的底层资源。

如下面例子，对应的 yaml 配置中，stage 字段可以从 .env 中读取配置；此外对应的资源名称中可以用 ${app}-${stage} 的方式动态命名。从而针对不同环境创建配置相同、相互隔离的资源。

![Serverless](https://img.serverlesscloud.cn/202078/1594192711986-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)

## 运维/排障

运维和排障一直是 Serverless 架构中客户反馈很多的问题，主要集中于以下两个方面：
- 缺乏应用级别的监控概览图，配置门槛高；
- 链路较长，不透明，故障难以排查。

针对这样的情况，腾讯 Serverless Framework 提供了开箱即用的应用级别监控视图，并且结合高级的日志查询功能，可以有效降低配置的学习成本，快速排障定位问题。

![Serverless](https://img.serverlesscloud.cn/202078/1594192711450-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)

## CI/CD
针对企业客户来说，接入接入自动化的 CI/CD 流程十分必要，主要有如下几个优势点：
- 减少重复操作，提升发布效率
- 降低风险，避免手动操作导致的故障
- 流程透明，充分的校验和测试

那么针对企业级客户连接 CI/CD 的诉求，Serverless Framework 既支持开源 CI/CD 产品的打通，如 Jenkins, Github Actions 等，也支持和 Coding 产品的一键打通，从而针对 Serverless 应用提供了“0”配置的 CI/CD 解决方案，实现构建、部署的流程的自动化。

![Serverless](https://img.serverlesscloud.cn/202078/1594192713173-%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_15941925279560.png)


最后，我相信 Serverless 的时代已经到来，它能够赋能开发者，助力企业上云，并将重新定义云的概念！

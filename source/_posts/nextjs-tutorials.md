---
title: SSR & Next.js 学习笔记
tags:
  - Nextjs
  - SSR
---

最近在调研 SSR 场景，并且在使用 Next.js 框架，特此记录分享。

## 概念说明：Next.js & SSR 

Server-Side-Rendering（SSR）泛指服务端渲染的技术，指的是在 Server 端将 HTML 渲染好，再返回给 Client 端。并且 SSR 是在对页面每个请求发出时，都会重新抓取和生成页面（和 SSG 静态页面生成相比，是更加动态的渲染方式）。

Next.js 是一个轻量级的 React 服务端渲染应用框架。支持多种渲染方式，包括客户端渲染、静态页面生成、服务端渲染。使用 Next.js 可以方便的实现 SSR，实现页面的服务端渲染。

## Next.js 支持的几种渲染方式

### 1. 客户端渲染 CSR Client Side Render
在浏览器上执行渲染，页面在浏览器获取 js 和 css 文件后渲染，路由也是客户端路由。
缺点：
- 白屏，加载时间长
- SEO 不友好，因为搜索引擎访问页面时默认不会执行 js，只看 html，所以看不到请求数据

但在一些场景中 CSR 也是适用的，例如有权限的（私有）页面，和 SEO 无关，并且页面内容经常更新，此时不需要预加载，直接通过 CSR 实现即可。

### 2. 静态页面生成 SSG Static Site Generation 

为了解决方法1 的问题，可以在后端进行一次渲染，浏览器直接请求，从而不需要每个客户浏览器端渲染。将动态内容静态化。
- 解决白屏和 SEO 问题
- 但这种方式对所有用户请求的内容都一样，无法生成用户相关内容

通过 `async` 方法 `getStaticProps` 获取内容，适用于静态页面生成过程中需要请求到外部数据的时候。例如页面中的信息要从 DB 中拉取，之后才能生成静态页。

<img width="500" alt="ssg" src="https://nextjs.org/static/images/learn/data-fetching/static-generation-with-data.png">

### 3. 服务端渲染 SSR Server Side Render

当页面和用户相关时，用 SSR 可以解决
- SEO 和白屏问题
- 生成用户相关内容，不同用户结果不同

通过 `getServerSideProps` 获取内容，在后端调用 `renderToString()` 的方法，把整个页面渲染成字符串。

附一张 CSR 和 SSR 和 SSG 的对比图，图片来源为参考文献中的文章。

![csr-ssr](https://img.serverlesscloud.cn/20201016/1602838536634-ssr-csr.png)

几种方案的优劣对比，图源详见参考文献。

![方案对比](https://img.serverlesscloud.cn/20201016/1602840854317-ssr.png)

<!--more-->

## 基于 Next.js 的博客系统搭建

了解了上述几种渲染方式后，可以通过实战来实现验证下。这里推荐下官方的博客搭建教程。通过非常详细的步骤进行部署，还有计分和分享系统。

### 1. Demo 效果

基于 Vercel 部署的 Next.js [博客主页](https://nextjs-blog-demo-tau.vercel.app/)

代码地址 [Github Repo](https://github.com/tinafangkunding/nextjs-blog-demo)

### 2. 功能说明

由于教程写的很详细了，这里仅简单列举一些博客搭建涉及的功能点：
 - [x] 搭建单页应用
 - [x] 页面之间相互导航
 - [x] Next.js 对静态资源，元数据和 CSS 的处理
 - [x] 预加载（SSR 和 SSG）及数据获取
 - [x] 动态页面的路由
 - [x] API 路由（Serverless 函数）
 - [x] 部署到 Vercel 平台
 - [x] 和 Github Actions 等 CI 打通

### 3. 部署方式

#### a. 部署到 Vercel，可以参考[教程](https://nextjs.org/learn/basics/create-nextjs-app)

#### b. 部署到腾讯云 Serverless SSR

1. 【前提】已经按照教程搭建了 Next.js 博客，在本地 localhost:3000 可运行，并有自己的 Github 仓库。
2. 【新建】登录腾讯云，打开 Serverless SSR [控制台](https://console.cloud.tencent.com/ssr)，如果是全新客户会有个授权的流程，授权完成后，点击新建应用，如下图所示。
![SSR 新建](https://imgbed-bucket-1251971143.cos.ap-guangzhou.myqcloud.com/./1604994564469-ssr-console1.jpg)
3. 【配置和导入】在新建页面中，填入博客项目名称，由于我本地已有部署好的 next.js 博客及仓库，因此可以直接选择「导入已有项目」。选择对应的代码托管方式，并进行一键授权。
![导入项目](https://imgbed-bucket-1251971143.cos.ap-guangzhou.myqcloud.com/1604995144841-ssr-console2.png)
配置完成后，点击部署，在「部署日志」页面查看和等待即可。
在这个过程中，Serverless SSR 会自动执行 CI 流程，做环境的初始化，安装 Serverless CLI，对项目进行 `npm run build` 构建，并且自动通过 layer 层对依赖进行分离，从而提升部署速度。

4. 【访问】等待约一分钟后，可以看到部署成功，跳转到了配置详情页面。此时点击对应的 URL 或者 「访问应用」 按钮，即可访问并打开博客了！
![访问页面](https://imgbed-bucket-1251971143.cos.ap-guangzhou.myqcloud.com/./1604995429926-ssr-console3.png)
至此，一行代码都没有改，我把博客无缝部署到了腾讯云 Serverless SSR 平台上托管。

## 参考文献
Next.js + TypeScript 搭建一个简易的博客系统
https://mp.weixin.qq.com/s/QM6MSsYK4cCv0i46TJExUw 

Next.js 官方教程-博客系统的搭建
https://nextjs.org/learn/basics/create-nextjs-app

Serverless SSR 在腾讯在线教育的实践
https://serverlesscloud.cn/best-practice/2020-05-22-edu-ssr-interview

对 Next.js 和 SSR 的讨论
https://arunoda.me/blog/hey-nextjs-is-server-side-rendering-dead

初探 Server-Side-Rendering 與 Next.js
https://oldmo860617.medium.com/%E5%88%9D%E6%8E%A2-server-side-rendering-%E8%88%87-next-js-%E6%8E%A8%E5%9D%91%E8%A8%88%E7%95%AB-d7a9fb48a964
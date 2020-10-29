---
title: Next.js 10 发布及特性解读
tags:
  - Nextjs
  - SSR
---

听了 10 月 27 日凌晨 Next.js Conf 大会的 Keynote，Next.js 10 发布了几个激动人心的特性，忍不住尝鲜体验下。

原文博客：
https://nextjs.org/blog/next-10

本次发布的特性主要聚焦于`开发者体验的提升`和`用户体验的提升`。

亮点目录：
1. 内置图片组件 Image Component : `next/image`，自动进行图片优化
2. 国际化路由 Internationalized Routing，多语言支持，自动语言检测
3. Next.js Analytics 性能测试，测试客户场景下真实性能
4. Next.js commerce 电商系统工具包 https://nextjs.org/commerce
5. SSR/SSG 渲染方法`getStaticProps / getServerSideProps` 快速切换 
6. React 17 的支持


## 1. 内置图片组件 Image Component，自动进行图片优化

当前网络上图片的占比有 50% 左右，图片的大小和尺寸很难控制（一半以上的图片都在 1M 以上）。一张比较大的图片，如果不经优化，在小尺寸设备，如手机端的显示并不好，也无必要。例如，2000 × 2000 像素的图片，在 100 × 100 的手机屏上显示就没有必要，并且加载速度慢。此外，就算研发和业务可以优化图片大小，但是对于客户上传的内容（CMS）也比较难控制和管理。

因此，Next.js 10 通过内置的图片组件 Image Component 解决这个问题，用法如下：

```js
import Image from 'next/image'

<Image src="/profile-picture.jpg" width="400" height="400">
```

通过 `next/image` 组件，图片会被自动 “懒加载(lazy-loaded)”，也就是只在用户快刷到图片时才加载。此外，限制了图片尺寸，防止布局偏移（layout shift)。
此外，image 组件将大尺寸图片自动裁剪和适配，生成适应屏幕尺寸的，WebP 格式的小图（WebP 比 JPEG 图小 30% 左右），这种情况下，即使是客户上传 CMS 中的图片内容也可以被自动裁剪和适配。

<!--more-->

## 2. 国际化路由 Internationalized Routing，多语言支持，自动语言检测

内置国际化路由和自动语言检测，默认支持荷兰语等。并且可以通过`子域名 or 子路径`的方式使用。

子路径路由例如 `/nl-nl/blog` 和 `/en/blog`，域名路由例如 `en.example.com` 和 `nl.example.com`，示例配置如下。

```js
// next.config.js
module.exports = {
  i18n: {
    locales: ['en', 'nl'],
    domains: [
      {
        domain: 'example.com',
        defaultLocale: 'en'
      },
      {
        domain: 'example.nl',
        defaultLocale: 'nl'
      }
    ]
  }
}
```
## 3. Next.js Analytics 性能测试，测试客户场景下真实性能

对页面进行持续的性能监测，而不是对页面进行一次性测试。
测试方案和指标是通过模拟用户实际行为/设备来收集的。对网站的评测更整体，更能体现客户的使用体验。Next.js Analytics 性能测试基于谷歌提出的 Core Web Vitals 指标，更贴近客户体验。例如客户加购物车后，购物车页产品数据同步展示的时间等。

## 4. Next.js commerce 电商系统工具包

基于疫情期间对电商站点的旺盛需求，并结合 Next.js 10 上述的特性，Next.js 还提供了一个 All-in-one 的高性能电商系统工具包，帮助开发者快速部署 Next.js 的电商网站。 https://nextjs.org/commerce

## 5. SSR/SSG 渲染方法动态切换

在 Next.js 10 中，编辑 `getStaticProps / getServerSideProps` 对应的函数时，Next.js 会自动重新执行函数使数据实时生效，从而不需要手动刷新页面。

## 6. React 17 的支持





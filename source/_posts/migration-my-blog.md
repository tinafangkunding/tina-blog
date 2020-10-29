---
title: 博客搬家记
tags:
  - Serverless
  - Hexo
  - Github Actions
---

现有博客的一些问题：
1. 手动构建和发布，比较繁琐，容易误操作
2. 依赖本地设备，换电脑的时候没法写博客了（电脑快被收了 > <）
3. 使用的部署工具是 serverless component v1，官方已经不再维护
4. 域名解析有点问题，每次都重定向到 cos 桶地址，并且在微信里被封禁，打不开，分享机制很不友好。

基于上述问题，终于决定迁移自己的博客了：
- [ ] 新的域名
https://blog.tinafang.com，解决问题 4
- [ ] 接入 Github Actions 的自动化 CI 流程，解决问题 1 和 2
- [ ] 升级为 component v2，解决问题 3
- [ ] 增加一些友情站点，yuga, only 的


## 步骤

1. 将本地代码提交到 github 远端上
2. 本地修改为 v2 的部署方式，配置新域名，并且手动部署成功，指定 cos-bucket（覆盖部署）
3. 配置 github actions，每次 git push 的时候自动 build 和 deploy


后面的计划：
- [ ] 搭建自己的图床
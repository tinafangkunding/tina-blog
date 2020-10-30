---
title: 博客搬家记
tags:
  - Serverless
  - Hexo
  - Github Actions
---

## 前言

由于种种原因，最近终于下定决心把博客重新部署下。整理了下现有博客的一些问题：
1. 手动构建和发布，比较繁琐，容易误操作
2. 依赖本地设备，换电脑的时候没法写博客了（电脑快被收了 > <）
3. 使用的部署工具是 serverless component v1，官方已经不再维护
4. 域名解析有点问题，每次都重定向到 cos 桶地址，并且在微信里被封禁，打不开，分享机制很不友好。

基于上述问题，终于决定迁移自己的博客了，当前的进展如下：
- [x] 新的域名
https://blog.tinafang.com，解决问题 4
- [x] 升级为 component v2，解决问题 3
- [x] 增加一些友情站点，例如 yuga, only 的
- [x] 兼容之前的博客地址，包括 cos 和 自定义域名
- [x] 传到 Github Repo 中，解决问题 2
- [ ] 接入 Github Actions 的自动化 CI 流程，解决问题 1
- [ ] 搭建自己的图床，对图片的大小，尺寸进行优化

## 步骤

### 1. 将本地代码提交到 github 远端上

1. 在gitHub 上 创建新库 `https://github.com/tinafangkunding/tina-blog`
2. 终端cd工程文件所在文件目录，
3. `git init`
4. `git add -A` (进入文件 添加文件,-A表示添加全部文件)
5. `git commit -m “提交说明”`
6. `git remote add origin url` (该url是你库的url)
7. `git push -u origin master` （将本地的仓库提交到github账号里）

刷新后，即可看到代码 https://github.com/tinafangkunding/tina-blog 

### 2. 升级为 component v2 的部署方式

1. 在本地环境把旧的 serverless.yml 改名，重建一个新的 serverless.yml
2. 并且将配置修改为 v2 规范：website component 的 v2 版本[配置参考](https://github.com/serverless-components/tencent-website)，也可以参考下文的配置。
3. 为新的 blog.tinafang.com 申请证书，半小时之后申请和验证都完成了。
4. 在 serverless.yml 中配置新域名和证书 id，指定原有的 cos-bucket（覆盖部署），并且手动部署成功。
5. 在 dns 中加一个 cname 记录。等待 1-2 分钟即可生效

此时，再次执行下列命令，更新到 github 仓库
```sh
git add -A
git commit -m "v2"
git push
``` 
<!--more-->

为了方便参考，这里贴一下 serverless.yml 的配置，并对其中的一些注意事项进行说明。

- 问题：部署了几次发现，每次更新文章，对应的 cos 域名内容很快更新，但自定义域名 blog.tinafang.com 中的内容刷新十分缓慢，后来发现是 CDN 有缓存的问题。
- 解决：async 参数配置为 false，auteRefresh 配置为 true， onlyRefresh 参数在首次部署后也配置为 true。再次编辑内容并部署，可以看到内容快速生效了！

1. **async 参数**：是否同步等待 CDN 配置。配置为 false 时，参数 autoRefresh 自动刷新才会生效，如果关联多域名时，为防止超时建议配置为 true。
2. **autoRefresh 参数**：配置开启自动 CDN 刷新，用于快速更新和同步加速域名中展示的站点内容。（该能力在 CDN 控制台也可以手动刷新，但是 website 组件贴心的集成了这个功能，赞）
3. **onlyRefresh 参数**：建议首次部署后，再将此参数配置为 true，即可以忽略其他 CDN 配置，只进行刷新操作

```yml
# serverless.yml
component: website # (必填) 引用 component 的名称，当前用到的是 tencent-website 组件
name: tinablog # (必填) 该 website 组件创建的实例名称
app: tinablogApp # (可选) 该 website 应用名称
stage: prod # (可选) 用于区分环境信息，默认值是 dev

inputs:
  src:
    #src: ./public
    index: index.html
    error: index.html
    dist: ./public   #这里 dist 和 hook 要一起用
    hook: npm run build
  region: ap-guangzhou
  bucketName: my-hexo-bucket
  protocol: https
  #cdn
  hosts:
    - host: blog.tinafang.com
      async: false # false 时自动刷新擦，true 的时候不会
      area: mainland
      autoRefresh: true
      onlyRefresh: true #true 时，cdn 只做刷新操作
      https:
        switch: on
        http2: on
        certInfo:
          certId: 'haMhGlIc'
``` 

### 3. Github Actions 配置（未完待续）

计划配置 github actions，每次 git push 的时候自动 build 和 deploy。但是当前发现 sls cli 中的 hook 命令，对于 hexo 来说，在远端好像不生效。需要生成秘钥才可以有权限。

Hexo + Git CICD 的参考文档（这里都是用 hexo cli 来做的，和我的需求不太一致）
https://www.jianshu.com/p/21dbc53a4f67
https://zhuanlan.zhihu.com/p/133764310
https://sanonz.github.io/2020/deploy-a-hexo-blog-from-github-actions/

Github Actions + Serverless 的参考文档 
https://cloud.tencent.com/document/product/1154/47290
https://blog.tinafang.com/2020/09/30/bilibili/

> 注：
1、远端创建了 workflow 的 main.yml 文件后，要记得本地 `git pull` 拉取一下最新的代码。之后再次更新对应的博客文件，然后 `git push`
2、github actions 需要 `sudo` 安装 serverless framework，否则会报权限不足的错误

## 小结和展望

博客搬家之前和之后，在部署流程，和访问者体验上有了提升。
- **部署流程**：之前需要 `hexo g` 之后 `sls --debug`，现在配置了 hook，只需要一步 `sls deploy`。但因为 CICD 没有打通，依然需要手动同步到 Github Repo
- **访问体验**：主要通过 https 的自定义域名解决了微信封禁的问题
<br/>
有空的时候会继续完善 Github Actions 的配置，并且搭建好图床。后面可以考虑针对当前网站里所有的图片进行优化和适配。另外当前自定义域名会有 cdn 缓存的问题，后面可以研究一下。
<br/>
项目地址：https://github.com/tinafangkunding/tina-blog/
博客新地址：https://blog.tinafang.com/
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

1.在gitHub 上 创建新库 https://github.com/tinafangkunding/tina-blog
2.终端cd工程文件所在文件目录，
3.git init
4.git add -A (进入文件 添加文件,-A表示添加全部文件)
5.git commit -m “提交说明”
6.git remote add origin url (该url是你库的url)
```
 git remote add origin https://github.com/tinafangkunding/tina-blog.git
```
7.git push -u origin master （将本地的仓库提交到github账号里）（可能会输入github的账号和密码） 

刷新后，即可看到代码 https://github.com/tinafangkunding/tina-blog 

2. 本地修改为 v2 的部署方式，配置新域名，并且手动部署成功，指定 cos-bucket（覆盖部署）

把旧的 serverless.yml 改名，重建一个新的 serverless.yml
修改配置为 v2 版本，配置参考：
https://github.com/serverless-components/tencent-website

之后，为新的 blog.tinafang.com 申请证书，半小时之后申请和验证都完成了。

重新部署，在 dns 中加一个 cname 记录。等待 1-2 分钟即可生效了！

此时，再次执行下 
git add -A
git commit -m "v2"
执行 $ git push 更新到 github 仓库

3. 配置 github actions，每次 git push 的时候自动 build 和 deploy

参考文档 https://cloud.tencent.com/document/product/1154/47290
https://blog.tinafang.com/2020/09/30/bilibili/

> 注：远端创建了 workflow 的 main.yml 文件后，要记得本地 `git pull` 拉取一下最新的代码。之后再次更新对应的博客文件，然后 `git push`
> 注2：github actions 需要 `sudo` 安装 serverless framework，否则会报权限不足的错误

## 小结和展望

博客搬家之前和之后，在部署流程，和访问者体验上有了提升。

- 部署流程：
- 访问体验：

后面的计划：
- [ ] 搭建自己的图床
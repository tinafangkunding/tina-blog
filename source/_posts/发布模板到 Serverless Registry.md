---
title: 发布模板到 Serverless Registry 
tags:
  - Serverless
  - Registry
---

看到 Tencent Serverless 最新发布了 Registry (可以翻译为模板商城，或者模板仓库)，并且提供了详细的[官方开发指南](https://github.com/serverless/components/blob/master/README.cn.md)。不仅想起之前创建一个 Python/PHP 的脚手架函数很麻烦，忍不住把几个常用的 hello world demo 发布上去尝鲜体验一下，特此记录。

## 前提条件

1. 下载安装了新版的 Serverless Framework 
```
npm install - g serverless
```
2. 拥有一个[腾讯云账号](https://cloud.tencent.com/login)

## 步骤

### 1. 查看可用组件和模板

查看可用模板中，可以看到主要列举的是一些官方的组件和模板

- 访问 Serverless 注册中心页面：https://registry.serverless.com
- 使用`sls registry`命令列出所有推荐的组件或者项目模板

发现官方的[模板仓库在这里>>](https://github.com/serverless-components/tencent-examples)

### 2. 开发上传自己的模板

准备好自己的代码项目，例如一个非常简单的 Python 3.6 Hello World。目录结构如下：
```
.
└── src
   |── index.py       # 入口函数
   └── serverless.yml # 依赖的组件配置说明，此处使用 scf 组件
```
此处的 serverless.yml 可以参考 scf 组件的全量配置。

为了让 Registry 识别到这是一个模板，需要在外层再创建一个 serverless.yml，并且描述一些模板相关的信息。此时，目录结构如下：

```
.
├── src
|   |── index.py       # 入口函数
|   └── serverless.yml # 依赖的组件配置说明，此处使用 scf 组件
└── serverless.yml # 模板说明文件
```
外层的 serverless.yml 描述如下：
```yaml
name: python-hello-world # 项目模板的名字
author: Tinafang # 作者的名字
org: Tencent Cloud, Inc. # 组织名称，可选
description: Deploy a Python 3.6 hello world demo. # 描述你的项目模板
keywords: tencent, serverless, python # 关键字
repo: https://github.com/serverless-components/python-hello-world # 源代码
readme: https://github.com/serverless-components/python-hello-world/tree/master/README.md # 详细的说明文件
license: MIT # 版权声明
src: # 描述项目中的哪些文件需要作为模板发布
  src: ./src # 指定具体的相对目录，此目录下的文件将作为模板发布
  exclude: #描述在指定的目录内哪些文件应该被排除
    - .env
```

本地部署验证成功，并且检查项目已经把秘钥文件排除之后，只需一行命令，就可以把模板发布到云端：
```
sls registry publish
```
发布成功，可以看到如下结果：
```
serverless ⚡ registry
Publishing "python-hello-world@0.0.0"...

Serverless › Successfully published python-hello-world
```
<!--more-->

那么，之后怎样查询对应的模板呢？在官方审核展示到列表和网页中之前，可以指定命名进行查询和使用：

查询已经发布的模板信息：
```
sls registry python-hello-world
```

### 3. 使用自己/官方的模板

为了验证模板是否已经发布成功，可以通过下列命令进行验证。首先可以通过初始化命令进行下载：

```
sls init -t python-hello-world
```
可以看到如下下载成功的结果：
```
serverless ⚡ framework

- Successfully created "python-hello-world" instance in the currennt working directory.
- Don't forget to update serverless.yml and install dependencies if needed.
- Whenever you're ready, run "serverless deploy" to deploy your new instance.

python-hello-world › Created
```

进到目录中，尝试把模板部署到云端：

```
cd python-hello-world && sls deploy
```

可以看到部署成功的信息。至此，不仅自己可以方便的创建一个 python demo，还可以分享给小伙伴一起使用，实在是让人忍不住贡献更多模板到 Registry 中！





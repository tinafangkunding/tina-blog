---
title: Serverless 实现 B 站打卡签到等（腾讯云函数部署教程）
tags:
  - Serverless
  - 应用
---

想要实现 B 站自动签到打卡，但是不想为闲置资源付费？

Serverless 让你不必买服务器，本地电脑也不用装 Python PHP 这些环境，全云端托管运行！

本文基于 [@happy888888/BiliExp](https://github.com/happy888888/BiliExp) 的代码，用 Github Actions 或 CLI 自动部署到腾讯云云函数上。

### 前提

1. 开通云函数 SCF 的腾讯云账号，在[访问秘钥页面](https://console.cloud.tencent.com/cam/capi)获取账号的 TENCENT_SECRET_ID，TENCENT_SECRET_KEY
> 注意！为了确保权限足够，获取这两个参数时不要使用子账户！需要提前开启云函数服务。此外，腾讯云账户需要[实名认证](https://console.cloud.tencent.com/developer/auth)。

2. 一个或多个B站账号，以及登录后获取的SESSDATA，bili_jct，DedeUserID (获取方式参考下图)

![获取 cookieDatas](https://img.serverlesscloud.cn/2020929/1601397752235-bili-1.png)
> 获取 cookieDatas（以 Chrome 为例）：登录 B 站 -> 右键点击「检查」-> application -> cookies

3. SCKEY (可选，用于账号失效时用微信提醒,不用请留空，详情见http://sc.ftqq.com/)

### 部署方案一：利用 Github Actions

1. fork Github 项目 https://github.com/happy888888/BiliExp

2. 在fork后的github仓库的 “Settings” --》“Secrets” 中添加"Secrets"，name和value分别为：
- name为"TENCENT_SECRET_ID" value为腾讯云用户SecretID(需要主账户，子账户可能没权限)
- name为"TENCENT_SECRET_KEY" value为阿里云账户SecretKey
- name为"biliconfig" value为B站账号登录信息，格式参照config/config.json文件
```json
{
     "cookieDatas":[
		{
			"SESSDATA": "",
			"bili_jct": "",
			"DedeUserID": ""
		}
	],
	"email": "",
	"SCKEY": "",
	"说明":"cookieDatas由浏览器获取，获取详情见首页说明；email用于邮件消息推送，SCKEY用于微信消息推送，详情见http://sc.ftqq.com/，这两项不用请留空"
}
```
环境变量添加完毕后如下图：

![环境变量](https://img.serverlesscloud.cn/2020929/1601398724122-bili-4.png)

3. 添加完上面 3 个"Secrets"后，进入"Actions" --》"deploy for tencentyun"，点击右边的"Run workflow"即可部署至腾讯云函数(如果出错请在红叉右边点击"deploy for tencentyun"查看部署任务的输出信息找出错误原因)。部署完成后的流程如下所示：

![部署完成效果](https://img.serverlesscloud.cn/2020929/1601398661220-bili-2.png)

> 注: 首次fork可能要去actions里面同意使用actions条款，如果"Actions"里面没有"deploy for tencentyun"，点一下右上角的"star"，"deploy for tencentyun"就会出现在"Actions"里面

<!--more-->

### 部署方案二：本地 CLI 部署

1. 安装命令行工具 Serverless Framework
```
npm install -g serverless
```

2. 下载项目模版代码，并进入模版目录 biliexp-demo
```
sls create --template-url https://github.com/happy888888/BiliExp.git
cd BiliExp
```

3. 打开 `config/config.json` 文档，根据说明填入对应内容，cookieDatas 由浏览器获取，email 处填入用于接受通知的邮件名
```json
{
     "cookieDatas":[
		{
			"SESSDATA": "",
			"bili_jct": "",
			"DedeUserID": ""
		}
	],
	"email": "",
	"SCKEY": "",
	"说明":"cookieDatas由浏览器获取，获取详情见首页说明；email用于邮件消息推送，SCKEY用于微信消息推送，详情见http://sc.ftqq.com/，这两项不用请留空"
}
```

4. 通过下列命令完成部署 (如果看部署详情可以加 --debug)，部署成功后，每日可自动触发，完成一系列操作
```
sls deploy
```

### 测试验证

部署到云函数的好处是，代码可控，便于修改和排查。

验证方式也很简单，进入到腾讯云[云函数控制台](https://console.cloud.tencent.com/scf/list?rid=1&ns=default)，可以看到成功部署的函数，点击”测试“可以看到函数调用成功，并输出相应的日志。

![测试脚本效果](https://img.serverlesscloud.cn/2020929/1601397992356-bili-3.png)

tina


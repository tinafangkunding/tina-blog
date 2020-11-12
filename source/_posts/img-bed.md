---
title: Serverless 搭建个人图床
tags:
  - Serverless
  - 应用
---

由于博客搬家，图片的存储来源也对应的替换了，所以参考了[微信公众号里的文章](https://mp.weixin.qq.com/s/dQZxBruMqCaSaUDxP5G1Hg)，按照上面的策略自建了图床。

实现上来说，就是把图片存到自己账户的对象存储 COS 里面，如果开启了自定义域名，则可以让图片路径也带有关键词，对于 SEO 会比较友好。

实现步骤很简单，如下所示：

1. 安装命令行工具 Serverless Framework
```
npm install -g serverless
```
2. 下载项目模版代码
```
sls init imgbed-for-scf
cd imgbed-for-scf/scf
```
在配置文件config.js 里填入您的 SecretId 与 SecretKey
```json
const config = {
    tencent_cos: {
        SecretId: 'XXXXXXXXXX', //您的 SecretId
        SecretKey: 'XXXXXXXXXXX', //您的 SecretKey
    }
}
module.exports = config
```
3. 部署
回到根目录下，执行 sls deploy 完成部署。
```
cd ..
sls deploy
```
4. 测试
部署成功后，打开 scf 目录下的 upload.html 文件，将创建成功的 API 网关 URL，填入 scf_url 字段里，之后打开该页面，就可以使用了。

为了便于访问，我还将 scf 目录下的 upload.html 也通过静态托管的方式存储在了 COS 里面，这样访问起来会更加方便。

最终效果如下
![imgbed](https://imgbed-bucket-1251971143.cos.ap-guangzhou.myqcloud.com/./1605174066708-imgbed.png)






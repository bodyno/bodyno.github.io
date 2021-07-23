---
layout: post
title: "用Serverless写一个视频下载器"
date: 2021-07-23
tags: [javascript]
categories: javascript
---

## 前言

越能体现人性尊严的快乐，越是一种最大的快乐，因为他跟人的尊严是相关的。有很多快乐，是降低了人性的尊严。而越能体现人性尊严的快乐，越是一种最高级的快乐，我们之所以读书行路，其实就是希望我们能够不断的。享受什么的快乐呢？高级的快乐。---- 罗翔老师经典语录


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/75ff6c4b6d5b4cdcae468f7313ca9f28~tplv-k3u1fbpfcp-watermark.image)

## 开始

今天看到一个开源库，用`Node.js`下载Youtube 视频，我眼前一亮，马上打开页面，看了一下介绍，好家伙，这么容易使用，那以后**打工人**上下班看离线视频那不是方便多了。我们先看一下API
https://github.com/fent/node-ytdl-core

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a112e7d13dad4d6ab8475a454f36776f~tplv-k3u1fbpfcp-watermark.image)

这API可真是简洁啊，我们把包在本地安装，然后下载不就好了。事情肯定没有这么简单，网络不通，哈哈。没关系~我有云，我有Serverless，我不用，就是玩。不不不，确实要用。

这里我的思路就是通过**API网关**生成一个URL，调用之后转到**云函数**，云函数下载视频到临时文件夹，云函数使用运行角色密钥上传视频文件至COS，我们再通过COS客户端在手机上面看视频。把函数部署到香港地域就没有网络的限制了，说着说着我都激动了一把。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fea631c50c241dc8149b702597346e0~tplv-k3u1fbpfcp-watermark.image)

大概思路就是图上这样。那我们开干吧。  
这里我选择**腾讯云**作为云提供商，因为腾讯云一直有免费的额度可以用，我们这一点点运算量，基本上不要米。技术上我们用`Node.js`，当然要上`TypeScript`这样才有逼格。  
我们通过Serverless这个框架创建一个`SCF`函数，然后基于它编写代码。

```js
$ npm i serverless -g
$ serverless init scf-demo
```
这样一个项目就创建好了
我们进入`src/index.js`下编写文件，先把后缀改为`.ts`，然后接着配置

```js
$ npm i ytdl-core cos-nodejs-sdk-v5 -S
$ npm i typescript ts-node prettier -D
```
这里`ytdl-core`是下载视频的包，`cos-nodejs-sdk-v5`是上传视频到COS的`Node SDK`

然后我们这里还需要一个强悍的打包工具，把所有代码打包到一起，毕竟云函数是不支持运行TS的。  
这里我推荐一个很好用的打包工具[ncc](https://github.com/vercel/ncc)，零配置，内置TS，按需加载，支持所有node包
```js
$ ncc build input.js -o dist
```
可以看到，打包方法非常简单，到时候我们把这个脚本配置到`npm build`命令里那不是美滋滋。我们全局安装一下
```js
$ npm i -g @vercel/ncc
```

## 编码

接下来我们编写代码：

```js
const fs = require('fs')
const path = require('path')
const ytdl = require('ytdl-core')
const COS = require('cos-nodejs-sdk-v5')

exports.main_handler = (event, context, callback) => {
  ytdl('https://www.youtube.com/watch?v=aqz-KE-bpKQ', {
    // 这里为了演示，只下载一小段，免得浪费大家流量
    range: {
      start: 0,
      end: 5355705,
    },
  })
    .pipe(fs.createWriteStream(path.join('/tmp', 'video.mp4')))
    .on('finish', () => {
      // 这里有个坑，临时密钥一定要填这个SESSION TOKEN 不然怎么都上传不了
      // 通过运行角色获取临时密钥
      const environment = JSON.parse(context.environment)
      const cos = new COS({
        SecretId: environment.TENCENTCLOUD_SECRETID,
        SecretKey: environment.TENCENTCLOUD_SECRETKEY,
        SecurityToken: environment.TENCENTCLOUD_SESSIONTOKEN,
      })

      cos.putObject(
        {
          Bucket: 'youtube-1253555942' /* 我们创建的COS Bucket */,
          Region: 'ap-hongkong' /* 地域 */,
          Key: `video-${Date.now()}.mp4` /* 文件名 */,
          StorageClass: 'STANDARD' /* 默认就好了 */,
          Body: fs.createReadStream(path.join('/tmp', 'video.mp4')), // 上传文件对象
          onProgress: function (progressData) {
            console.log(JSON.stringify(progressData))
          },
        },
        function (err, data) {
          console.log(err || data)
          callback(null, 'ok')
        },
      )
    })
    .on('error', (e) => {
      console.log(e)
    })
}

```
这段代码的逻辑：引入包，进入主函数`main_handler`，编写下载逻辑，存入临时文件，创建COS对象，上传COS文件，结束。一气呵成，非常顺滑。TS的类型就交给你们补充了。云函数支持[两种异步方法](https://cloud.tencent.com/document/product/583/11060#node.js-10.15-.E5.8F.8A-12.16-.E7.9A.84.E5.BC.82.E6.AD.A5.E7.89.B9.E6.80.A7)，一种async，一种callback，这里我们用到了流所以`callback`的方式会简单一些，调用callback告诉引擎函数执行完毕。  
我们再配置一下项目就可以运行起来了。首先是`serverless.yml`这个是上传至云函数的配置，这里用到香港地域，还有代码路径填我们打包后的目录，这样可以少上传些文件，让部署速度更快，其它配置基本不变。

```yaml
#应用信息
app: scf-demo-3723d4bb

#组件信息
component: scf # (必填) 引用 component 的名称，当前用到的是 tencent-scf 组件
name: youtube # (必填) 创建的实例名称，请修改成您的实例名称

#组件参数
inputs:
  name: ${name}-${stage} #函数名称
  src: ./dist  #代码路径
  handler: index.main_handler #入口
  runtime: Nodejs10.15 # 云函数运行时的环境
  region: ap-hongkong # 云函数所在区域
  events: # 触发器
    - apigw: # 网关触发器
        parameters:
          endpoints:
            - path: /
              method: GET

```

再配置一下我们的`deploy`脚本，这样我们运行`npm run deploy`时，会先执行打包再部署。

```js
"scripts": {
    "start": "ts-node src/local.ts",
    "build": "ncc build src/index.ts",
    "deploy": "npm run build && sls deploy"
  },
```
一切准备就绪，运行`npm run deploy`，可以看到已经成功上传到云函数了。并且还帮我们创建了一个**网关触发器**，可以通过URL直接触发访问函数

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18fde7e92bdb4081a6470f82b9101ef7~tplv-k3u1fbpfcp-watermark.image)


![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29ca0d80c2e0457496101a28b7555bb2~tplv-k3u1fbpfcp-watermark.image)

我们登上函数控制台页面，在函数配置这里把运行角色给加上，后续上传到COS，就可以直接从函数运行上下文中取到SecretKey和SecretID了，不用写死在代码里面，很方便。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0997f9b77544e6aba076521a4ae818b~tplv-k3u1fbpfcp-watermark.image)

我们手动测试一下函数

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9929e3ccdadb418eb17c8ffd0d0f0d64~tplv-k3u1fbpfcp-watermark.image)

不错，成功了


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3eb8eb91d0eb42308c35204de576d53b~tplv-k3u1fbpfcp-watermark.image)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa1b9e3812e5489583ee20253fe15275~tplv-k3u1fbpfcp-watermark.image)

可以看到COS里已经有我们想下载的视频文件了，xdm是不是很方便。  
这里我偷个懒，通过网关URL参数获取视频地址，这个功能就交给你们开发了。  
我把代码提交一下，大家可以直接Fork到自己仓库下开发。

[serverless-youtube-download](https://github.com/bodyno/serverless-youtube-download)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/545c185a8b8c46a8add3f76e95710bd1~tplv-k3u1fbpfcp-watermark.image)

## 再见

感谢大家支持，我们下期再见。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/849922d87cfb4ea89f42a7548d4c3c36~tplv-k3u1fbpfcp-watermark.image)

---
layout: post
title:  "语义化的版本号控制"
date:   2016-05-09
tags: [npm]
categories: Front-End
---

使用bower和npm的时候进行依赖管理的时候，需要创建`bower.json`和`package.json`。

此处以`package.json`为例。

运行以下命令会将依赖项的版本信息写入`bower.json`或者`package.json`中：

{% highlight bash %}
$ npm install PACKAGE_NAME --save
{% endhighlight %}

一个`package.json`可能会是下面这个样子：

{% highlight json %}
{
  "name": "example-app",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "api_500px": "0.0.3",
    "body-parser": "^1.13.3",
    "cookie-parser": "^1.3.3",
    "express": "^4.13.4",
    "jquery": "~2.1.4",
    "gulp": "*"
  },
  "devDependencies": {}
}
{% endhighlight %}

在这个文件里，有三种版本号的写法：`^`，`*`和`~`。如果要使用npm来对这些以来进行升级，这些符号就起作用了。

> 版本号的格式为`主版本号.次版本号.补丁次数`

假设有一个依赖的版本为`1.0.1`，那么，根据这三种写法，更新依赖的时候，会按照以下的方式进行更新。

**1、使用`~`的版本号**

对于使用`~`的版本号，更新的时候，只会更新它的补丁版本号。

在`package.json`里写入版本号或者安装依赖时指定版本的时候，可以写成`1.0`，`1.0.x`或者`~1.0.1`。

**2、使用`^`的版本号**

对于使用`^`的版本号，更新的时候，只会更新它的次版本号。

在`package.json`里写入版本号或者安装依赖时指定版本的时候，可以写成`1`，`1.x`或者`^1.0.1`。

**3、使用`*`的版本号**

对于使用`*`的版本号，更新的时候，主版本号也会更新。

在`package.json`里写入版本号或者安装依赖时指定版本的时候，可以写成`x`或者`*`。

就是这样 喵~

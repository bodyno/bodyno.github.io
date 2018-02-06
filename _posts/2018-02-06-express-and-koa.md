---
layout: post
title: "关于：Express会被Koa2取代吗？"
date: 2018-02-06
tags: [node]
categories: programe
---

知会上看到有个问题[《Express会被Koa2取代吗？》](https://link.juejin.im/?target=https%3A%2F%2Fwww.zhihu.com%2Fquestion%2F266464363%2Fanswer%2F312290658)。刚好对Express、koa有点小研究，于是简单回答了一下。

### 1、先说结论

#### 目前没有看到Express会被koa2取代的迹象。

目前来说，Express的生态更成熟，入门门槛相对较低。从npm上的下载热度来说，两者的差距还较大，Express的月下载量约为koa2的40倍。

不过koa2的亮点足够吸引人，生态也开始变得完善。

### 2、从使用门槛来说

从使用上来说，Express对初学者更有好些，对着官网修修改改改就能做点东西出来。

koa2入门门槛比Express高些。更精简的内核带来的小问题就是，对使用者搭积木的能力要求更高了，毕竟连核心的路由功能都去掉了。

更不要说koa2中最吸引人的async/await，很多初学者promise都搞不明白，async/await用起来一头雾水，koa2最精华的部分之一就派不上用场了。

### 3、从大趋势来说

node社区壮大后，参与node服务端编程的同学会越来越多。届时，对服务端框架的要求会越来越高，那个时候就是各种企业级解决方案们的战场了。核心很有可能还是基于Express或者koa2，或者其他的。

至于Express和koa2，还是会继续有很大的市场，那个时候版本不知道是多少。

### 4、后话

Express、koa2略有小研究，最近刚撸了一遍源码。后续会继续分享Express或koa2周边相关的技术文章 :-)
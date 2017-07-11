---
layout: post
title: "Vue nextTick"
date: 2017-07-11
tags: [programmer]
categories: programmer
---

Vue nextTick是一个非常有用的方法，在项目中，我们要经常改变一个变量的值，然后获取到改变的dom节点，这个时候，如果直接使用同步的编程方式去获取这个dom节点，是获取不到的，会报错。

在不知道这个方法之前，我用的是一个定时器10ms一次去找这个dom节点，这样虽然也可以实现功能，但是，豪无优雅性可言。

当我看到`this.$nextTick()`这个方法的时候真的是眼前一亮，这样代码不就优雅了吗？

下面是官方的说明

![vue nextTick](https://dn-coding-net-production-pp.qbox.me/3ae06072-8a79-456e-8082-d0c50ef54123.png)


再赋上我们的代码

````
this.$nextTick(function () {

})
// promise化
this.$nextTick().then(() => {

})
````

小伙伴们看一下，这样写一下是不是就优雅很多了

下次有空再教大家一些vue的小技巧
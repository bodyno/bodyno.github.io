---
layout: post
title: "一个Vue的小技巧"
date: 2018-01-21
tags: [vue]
categories: programe
---

近期年底了，有点忙，过段时间写一个2017年总结，也算是对2017年的一个交代。

这里分享一个关于Vue图片懒加载的小技巧，


````
<lazy-image :src='../assets/rey.jpg' />
````

我们知道想要懒加载图片是希望页面显示出来之后再去加载这张图片，

所以当webpack打包前，我们是不知道图片打包后的路径或文件名的，

那我们就需要动态引入图片的路径，让webpack打包完成后，将路径和文件名替换掉。


````
<lazy-image :src='require("../assets/rey.jpg")' />
````

这样，就可以完成图片懒加载并webpack打包了，不用像以前的老方法，先上传到oss上面再来引用。

![result](/assets/vue_result.png)

是不是感觉一下开朗了许多，

以后有类似的小技巧我也会及时分享出来的。
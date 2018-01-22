---
layout: post
title: "一个并排排列不换行，无需设置宽度的方法"
date: 2018-01-22
tags: [css]
categories: programe
---

今天，在浏览某网站的时候，发现和我经常做的列表横向滚动的需求一样，

于是乎，手机打开链接，分享到文件传输助手，chrome打开网页，debug开始。

一眼看上去，和我常用的方法并不一样，甚至比我的方法更优雅。

经过简单的研究，很快就找到了其中的秘密。

直接上demo

HTML


````
<div class='con'>
  <div class='list'>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
    <div class='item'></div>
  </div>
</div>
````

CSS

````
.con {
  overflow: hidden;
  overflow-x: auto;
  width: 300px;
}
.list {
  white-space: nowrap;
}
.item {
  width: 50px;
  height: 50px;
  background: violet;
  display: inline-block;
}
````

仔细看，`.list`的样式`white-space: nowrap;`配合上`.item`的样式`display: inline-block;`就可以达到效果。

上效果图

![break](/assets/break_line.png)

嗯，是不是不用设置宽度了。

有时候看到别人网页，不妨打开看一看，研究一下，有意想不到惊喜也说不到，正所谓：三人行，必有我师焉。

今天的简单分享就到这里。
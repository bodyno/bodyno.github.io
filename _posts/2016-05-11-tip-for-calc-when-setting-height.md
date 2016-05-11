---
layout: post
title:  "CSS3的calc属性设置高度的时候的tip"
date:   2016-05-11
tags: [css3]
categories: Front-End
---

CSS3有一个很强大的`calc`属性可以供我们通过动态计算的方法来设置元素的宽高。

但是，在使用这个属性的时候，需要有一些注意的地方。

看下面这个例子：

{% highlight html %}
<div class="container">
    <div class="title">Title</div>
    <div class="content">Content</div>
    <div class="footer">Footer</div>
</div>
{% endhighlight %}

{% highlight css %}
.title {
  width: 100%;
  height: 100px;
  background-color: blue;
}

.content {
  width: 100%;
  
  /* 设置内容高度为视区的总高度减去footer和header的高度，使用CSS3的calc属性 */
  height: calc(100% - 150px);
  background-color: #aaaaaa;
}

.footer {
    position: fixed;
    width: 100%;
    height: 50px;
    bottom: 0;
    background-color: black;
}
{% endhighlight %}

表面上看这段代码没有问题，但是，我们在使用的时候就会发现，高度没能设置成功。因为高度为100%的时候，是相对于父级容器的高度来的，所以，还需要给父级容器设置高度才行。

所以，更改后的CSS如下：

{% highlight css %}
/* 父级容器设置高度为视区高度，vh表示view height，100vh表示高度为100% */
.container {
    color: #ffffff;
    height: 100vh;
}

.title {
  width: 100%;
  height: 100px;
  background-color: blue;
}

.content {
  width: 100%;
  
  /* 设置内容高度为视区的总高度减去footer和header的高度，使用CSS3的calc属性 */
  height: calc(100% - 150px);
  background-color: #aaaaaa;
}

.footer {
    position: fixed;
    width: 100%;
    height: 50px;
    bottom: 0;
    background-color: black;
}
{% endhighlight %}

这样，我们的高度设置就可以了。

谢谢大家。
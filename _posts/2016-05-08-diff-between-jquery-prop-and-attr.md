---
layout: post
title:  "jQuery中的prop和attr的区别"
date:   2016-05-07
categories: Front-End
---


在jQuery中, 获取元素的属性值有两种方法, 分别是`.prop()`和`.attr()`, 那么, 这两者的区别是什么呢?

jQuery官方有一段文字是这样解释的:

在一些特殊的情况下，attributes和properties的区别非常大。在jQuery1.6之前，.attr()方法在获取一些attributes的时候使用了property值，这样会导致一些不一致的行为。在jQuery1.6中，`.prop()`方法提供了一种明确的获取property值的方式，这样`.attr()`方法仅返回attributes。

比如, 以下类型的属性应该使用`.prop()`来设置以及获取:

`selectedIndex`, `tagName`, `nodeName`, `nodeType`, `ownerDocument`, `defaultChecked`, and `defaultSelected`。

这样说不好理解, 举个例子。

假设我们的HTML是下面这一句:

{% highlight html %}
<input type="checkbox" id="testBox" name="boxName">
{% endhighlight %}

现在, 我们要获取这个`checkbox`的属性值。有两种方式, 那么, 这两种方式都能够获取到吗?

{% highlight javascript %}
/* 获取checked属性 */
console.log($('#testBox').attr('checked')); // undefined
console.log($('#testBox').prop('checked')); // false

/* 获取name属性 */
console.log($('#testBox').attr('name')); // "boxName"
console.log($('#testBox').prop('name')); // "boxName"

/* 获取myName属性 */
console.log($('#testBox').attr('myName')); // "Erichain"
console.log($('#testBox').prop('myName')); // undefined
{% endhighlight %}

这样, 应该就能够看出一些区别了吧。

可以总结为以下几点:

1. 当要获取元素的固有属性的时候,使用`.prop()`方法, 比如`checked`, `name`等属性;

2. 当要获取元素的自定义属性的时候, 使用`.attr()`方法, 比如自己定义的`data-*`属性, 前面例子中的`myName`属性等;

3. 当获取的属性值为`true`或者`false`的时候, 使用`.prop()`方法。
---
layout: post
title:  "在CoffeeScript中定义私有成员变量"
date:   2016-07-05
tags: [coffeescript]
categories: Front-end
---

最近花了一些时间把CoffeeScript学习了一下, 说实话, 习惯了原生Javascript的语法和格式, 对于Coffee还真有点不太适应: 一是在Coffee里基本上都不会去写分号和括号, 大括号啥的(对于一个有分号强迫症的人来说, 这还真得适应一段时间); 然后就是, 使用类似于Python的那种语法格式, 通过代码缩进来让编译器进行推导.

不过, 本文的主要目的当然不是不是吐槽Coffee, 而是我在看文档的过程中, 遇到了一个问题, 查阅了一些资料后, 还是解决了我心中的疑惑, 于是便拿出来分享一下.

在Coffee的官方文档中, 有下面这样一段代码(我截取了其中的一小部分, 在文档的[Classes, Inheritance, and Super部分](http://coffeescript.org/#classes)):

{% highlight coffeescript %}
class Animal
    constructor: (@name) ->

    move: (meters) ->
        alert @name + " moved #{meters}m."
{% endhighlight %}

这段Coffee所对应的Javascript如下(我也只是截取了一小部分):

{% highlight javascript %}
Animal = (function() {
    function Animal(name) {
        this.name = name;
    }

    Animal.prototype.move = function(meters) {
        return alert(this.name + (" moved " + meters + "m."));
    };

    return Animal;
})();
{% endhighlight %}

那么问题来了. `constructor`是Animal类的构造函数, 这个没什么问题, 然后, 在Animal这个类里, 定义了一个`move`方法, 按理说这个方法属于Animal类. 当然, 确实属于. 但是, 奇怪的是, 为什么它会默认添加到Animal的`prototype`上去呢?

一开始, 我以为是`constructor`在作怪, 就把`constructor`这一块去掉了, 就像这样:

{% highlight coffeescript %}
class Animal
    move: (meters) ->
        alert @name + " moved #{meters}m."
{% endhighlight %}

但事实证明, 我的想法是完全错误的. 这段Coffee所编译出来的Javascript中, `move`方法还是在`Animal.prototype`上.

然后, 我在`move`方法下面又定义了一些变量和方法, 但无一例外, 全部都附加到了`Animal.prototype`上.

于是, 我上Google去查了一下, 看到了一篇比较好的文章说了这个问题: [Using private methods in CoffeeScript](https://crimefighter.svbtle.com/using-private-methods-in-coffeescript).

那么, 如果要在Coffee中定义私有成员变量的话, 该怎么做呢?

其实很简单, 就像下面这样:

{% highlight coffeescript %}
class Animal
    constructor: (@name) ->

    publicMethod: ->
        'this is a public method'

    privateMethod = ->
        'this is a private method'

    privateVariable = 'private'
{% endhighlight %}

所以, 不难看出来, 在Coffee中定义公有成员和私有成员的差别就是使用`:`还是`=`. 所以, 通过使用`=`, 我们就可以在Coffee中为类定义私有成员变量了.
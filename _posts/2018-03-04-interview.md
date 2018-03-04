---
layout: post
title: "聊聊一道简单的javascript面试题"
date: 2018-03-04
tags: [javascript]
categories: programe
---

下面是一道很入门的js面试题:

    for (var i = 0; i < 10; i++) {
      setTimeout(function () {
        console.log(i)
      }, 10 * i)
    }


几乎每个前端在初学js的都会遇到这个问题, 有一段时间也是面试必问的题, 当然现在看到这段代码几乎不用想, 输出肯定是`10*10`.

原因也是很简单: 变量提升. js没有块级作用域, 所以在for循环中定义的i提升为全局的了, 另外for循环是同步执行的, 所有当`setTimeout`内部的匿名函数执行的时候i已经是10了.

那怎么解决呢? 也没啥疑问, 闭包或者用let:

    // 闭包
    for (var i = 0; i < 10; i++) {
      void function (j) {
        setTimeout(function () {
          console.log(j)
        }, 10 * j)
      }(i)
    }

    // let
    for (let i = 0;  i < 10; i++) {
      setTimeout(function () {
        console.log(i)
      }, 10 * i)
    }


为什么上面的方法能解决呢? 闭包那个不用多说, 因为js有函数作用域. i作为参数传入, 直接绑定到匿名函数上, 作用域链到此截至, i再提升也跟他没关系了. 至于let, 其实是js内部实现的问题了, 简单讲就是let会生成不同的i实例, 10个匿名函数其实分别得到的是10个不同的i实例, 最终获取的当然是理想值了.

当然这篇文章不会在这里简单的结束. 我们再深入一点, 来看看第一段代码的执行过程把:

![image](/assets/interview_1.jpg)

我们可以看到, for 每执行一次, 就会调用setTimeout延迟若干秒向事件队列推入一个匿名函数, 但是因为for是同步的, 所以推入的匿名函数不是立马执行的, 而是要等for循环结束, 当然for执行时间很快, 但是影响却不小, 由于作用域问题, i被提升了, 当for结束了全局i就是10, 这时候call stack也空了, 匿名函数开始依次推入到call stack执行, 由于引用的都是变量i, 而i已经是10了, 输出`10*10`没毛病. 如果用let呢? 根据[mdn](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FStatements%2Flet), 每次for都会创建一个新的i binding, 也就是说匿名函数引用的是不同的i实例. 结果不言而喻.

![image](/assets/interview_2.jpg)

所以, 一个简简单单的面试题还是有很多可挖掘的点, 比如上面我们就涉及了作用域, 异步同步, 事件循环等等, 每个点都可以深入说很多: let const的暂存死区, 异步的执行顺序, 事件循环的基本实现等等. 希望有机会可以深入讨论.

---
layout: post
title: "如何解决 touchstart 事件与 click 事件的冲突"
date: 2018-03-08
tags: [javascript]
categories: programe
---

一 · 业务场景的描述
-----------

在对已完成的PC站点进行移动端适配时，我们想要站点在移动设备上有更快的响应速度，以带给用户更好的体验，此时，我们应该使用移动设备专用的事件系统，例如，使用 `touchstart` 事件代替 `click` 事件。

为什么这样效果会更好呢？根据Google开发者文档中的描述：

> mobile browsers will wait approximately 300ms from the time that you tap the button to fire the click event. The reason for this is that the browser is waiting to see if you are actually performing a double tap.

移动设备上的浏览器将会在 `click` 事件触发时延迟 **300ms** ，以确保这是一个“单击”事件而非“双击”事件。

而对于 `touchstart` 事件而言，则会在用户手指触碰屏幕的一瞬间触发所绑定的事件。所以，使用 `touchstart` 替换 `click` 事件的意义在于，帮助用户在每次点击时节省 **300ms** 的时间。在页面频繁需要点击，或者点击发生在动画中，对动画流畅度有较高要求的情境下，使用这种技术是非常必要的。

但是，让我们回到我们的初始场景，在 **PC端站点适配移动端时** 我们不能简单的进行 `touchstart` 和 `click` 事件的替换，因为PC并不能识别 `touchstart` 事件。

二 · 产生冲突的原因
-----------

当然，我们可以给某个元素同时绑定 `touchstart` 和 `click` 事件，但这将会导致本篇文章解决的问题 \-\- **这两个事件在移动设备上会发生冲突**。

由于移动设备能够同时识别 `touchstart` 和 `click` 事件，因此当用户点击目标元素时，绑定在目标元素上的 `touchstart` 事件与 `click` 事件（约300ms后）会依次被触发，也就是说，**我们所绑定的回调函数会被执行两次！**。这显然不是我们想要的结果。

三 · 解决方案
--------

针对这样的情境，有以下两种解决方案：

### （一）使用 preventDefault

第一种解决方案是使用事件对象中的 `preventDefault` 方法，对于该方法MDN上的解释是：

> The Event interface's preventDefault() method tells the user agent that if the event does not get explicitly handled, its default action should not be taken as it normally would be.

可见， `preventDefault` 方法的作用在于：**阻止元素默认事件行为的发生**，但有意思的是，当我们在目标元素同时绑定 `touchstart` 和 `click` 事件时，在 `touchstart` 事件回调函数中使用该方法，可以阻止后续 `click` 事件的发生。

这从道理上是讲不通的，毕竟，我们添加的 `click` 事件并不是元素的“默认事件”，但它确实奏效了，或者说，被浏览器实现了，因此我们可以使用该方法解决移动设备上 `touchstart` 事件与 `click` 事件的冲突问题，具体代码如下：

    const Button = document.getElementById("targetButton")

    Button.addEventListener("touchstart", e => {
        e.preventDefault()
        console.log("touchstart event!")
    })

    Button.addEventListener("click", e => {
        e.preventDefault()
        console.log("click event!")
    })


当你在浏览器上模拟移动设备后点击目标元素，只会在控制台看到 `touchstart event!` 字段，很显然，`click` 事件被成功阻止了。

**总结**

使用该方法的优点在于简单粗暴，直接有效，能够很好的实现我们的目标，但缺点在于， `preventDefault` 方法为阻止 `click` 事件的方式是浏览器实现上的，而不是 `preventDefault` 原理上的，这会带来一些不确定性，虽然我暂时尚未发现该方法失效的具体场景。

###（二）基于功能检测绑定事件

第二种方法是受到这篇[博客](https://link.juejin.im?target=https%3A%2F%2Fjoshtronic.com%2F2015%2F04%2F19%2Fhandling-click-and-touch-events-on-the-same-element%2F)的启发，我们可以通过判断浏览器是否支持 `touchstart` 事件来封装元素的点击事件，这样客户端会根据当前环境判定元素应该绑定的事件类型，代码如下：

    const Button = document.getElementById("targetButton")

    const clickEvent = (function() {
      if ('ontouchstart' in document.documentElement === true)
        return 'touchstart';
      else
        return 'click';
    })();

    Button.addEventListener(clickEvent, e => {
      console.log("things happened!")
    })


**总结**

该方法的优点在于，我们通过增加一次判断，为元素减少了一个不必要的事件绑定，从而避免了 `touchstart` 与 `click` 事件的冲突问题。这种方法避免了我们书写两次同样的代码，并且相较于第一种方法更加符合逻辑，因此是我所推荐的。
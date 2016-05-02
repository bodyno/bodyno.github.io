---
layout: post
title:  "理解Angular中的$digest()和$apply()"
date:   2016-05-02
categories: Angular Front-End
---

翻译自[http://www.sitepoint.com/understanding-angulars-apply-digest/](http://www.sitepoint.com/understanding-angulars-apply-digest/)

`$digest()`和`$apply()`是AngularJS中的两个核心并且有时候容易引人误解的部分。我们需要深入理解这两者是如何运作的，从而才能理解AngularJS本身是如何运作的。本文的目的就是为了和你解释，在你的日复一日使用AngularJS编写代码的过程中，`$digest()`和`$apply()`是如何确确实实的对你有用的。

### `$digest()`和`$apply()`的探索

AngularJS为我们提供了一个广为人知的令人惊叹的功能，那就是数据的双向绑定。这个功能极大的方便了我们的编码。数据绑定意味着，当视图(view)中的数据发生变化的时候，作用域下的数据模型(model)也会相应的更新。同样的，无论何时，当数据模型(model)发生变化的时候，视图(view)也会相应的更新。那么，AngularJS是怎么做到的呢？当你写这样的一个表达式的时候( {{ aModel }} )，AngularJS在背后为这个数据模型(model)设置了一个监听器(watcher)，正是由于这个监听器(watcher)，无论何时当数据模型(model)发生变化的时候，视图(view)也会更新。这个监听器(watcher)与你在AngularJS中设置的其它监听器类似(watcher)：

{% highlight javascript %}
$scope.$watch('aModel', function ( newValue, oldValue ) {
	// update the DOM with newValue
});
{% endhighlight %}

大家都知道，传入`$watch()`的第二个参数是一个监听函数，并且，无论何时当`aModel`的值发生改变的时候，他都会被调用。对于我们来说，这样理解很简单，当`aModel`发生改变的时候，这个监听函数被调用，然后，更新HTML中的表达式的值。但是，有一个很大的问题在于，AngularJS是怎么知道什么时候去调用这个监听函数呢？换句话说，AngularJS是怎么知道`aModel`发生变化的从而去调用相对应的监听函数呢？难道是周期性的运行一个函数来检测数据模型是否发生了变化吗？好吧，这时候，就需要讲到`$digest`循环了。

监听器(watcher)是在`$digest`循环中被启用的。当一个监听器(watcher)被启用的时候，AngularJS就会去评估数据模型(model)，并且，当数据模型(model)的值发生改变的时候，就会去调用相应的监听函数。所以，下一个问题就是，`$digest`循环是怎么开始的。

当运行`$scope.$digest()`的时候，`$digest`循环就开始了。假设你通过`ng-click`指令来调用处理函数来改变一个数据模型(model)，在这种情况下，AngularJS就会自动调用`$digest()`触发`$digest`循环。当`$digest`循环开始的时候，他就会启动每一个监听器(watcher)。这些监听器(watcher)会去检查当前的数据模型(model)中的值是否与最后一次计算的值相同，如果不相同，那么，对应的监听函数就会被执行。结果就是，如果你的视图(view)中有表达式，那么，这些表达式就都会被更新。除了`ng-click`，还有一些其它的内置指令(或者服务)可以让你改变数据模型(比如`ng-model`，`$timeout`等)，并且自动触发`$digest`循环。

到目前为止，很棒有没有？但是，仍然有一些小的不足。在上面的例子中，AngularJS并不是直接调用`$digest()`，而是通过`$scope.$apply()`然后，相应的调用`$rootScope.$digest()`。这样做的结果就是，一个`$digest`循环在`$rootScope`开始，随后，逐步的遍历所有的子作用域来调用监听器(watcher)。

现在，假设你给一个按钮添加了`ng-click`指令，并且为他传递了一个函数。当你点击这个按钮的时候，AngularJS会把这个函数放进`$scope.$apply()`中进行调用。所以，你的函数能够正常的执行，改变数据模型(如果有的话)，并且，`$digest`循环会开始来保证你的更改会反映到视图(view)中。

> 注意：`$scope.$apply()`会自动调用`$rootScope.$digest()`。`$apply()`函数可以以两种方式运行。第一种是将函数作为参数，并且评估他，然后触发`$digest`循环。第二种并不传入任何参数，仅仅是执行一个`$digest`循环。我们将会知道为什么前一种方法是更好更直接的方法。

### 什么时候需要人为的调用`$apply()`呢

如果AngularJS总是将我们的代码放在`$apply()`中并且执行`$digest`循环，那么，什么时候需要我们人为的调用`$apply`呢？事实上，AngularJS已经很明确的告诉我们了。AngularJS只会关心在AngularJS的执行上下文中的发生的数据模型(model)的变化(比如，改变数据的代码在`$apply()`里面)。AngularJS内建的指令也会自动触发`$digest`循环所以任何数据模型(model)的改变都会反映到视图中。但是，如果我们更改一个不在AngularJS执行上下文中的数据模型(model)，就需要人为的调用`$apply()`来提醒AngularJS数据发生变化了。就像是要告诉AngularJS，我们改变了一些数据，他应该启用监听器以便于让我们所做的改变能够反映出来。

例如，当使用Javascript的`setTimeout()`函数来更新一个数据模型的时候，AngularJS就没办法知道你改变了数据模型。这种情况下，调用`$apply()`来触发`$digest`循环就是你的责任了。类似的，如果你写了一个指令，这个指令是设置了一个DOM事件监听器，更改数据模型的代码在事件处理函数里，那么，也需要调用`$apply()`来保证更改能够反映出来。

让我们来看个例子。假设有一个页面，我们想在页面加载完之后两秒显示一个信息。我们可能这样写：

{% highlight html %}
<body ng-app="myApp">
    <div ng-controller="MessageController">
        Delayed Message: {{message}}
    </div>
</body>
{% endhighlight %}

{% highlight javascript %}
/* 没有$apply()会发生什么 */
angular.module('myApp', []).controller('MessageController', function ( $scope ) {

    $scope.getMessage = function () {
        setTimeout(function () {
            $scope.message = 'Fetched after 3 seconds';
            console.log('message:'+$scope.message);
        }, 2000);
    };

    $scope.getMessage();
});
{% endhighlight %}

通过运行这个例子，我们会发现，在两秒之后，函数开始执行，并且更新数据模型。但是，视图却没有更新。原因我们也应该猜到了，我们忘了调用`$apply()`。因此，我们需要像下面这样来写我们的`getMessage()`函数。

{% highlight javascript %}
/* 使用了$apply()会发生什么 */
angular.module('myApp', []).controller('MessageController', function ( $scope ) {
    $scope.getMessage = function () {
        setTimeout(function () {
            $scope.$apply(function () {
                //wrapped this within $apply
                $scope.message = 'Fetched after 3 seconds';
                console.log('message:' + $scope.message);
            });
        }, 2000);
    };

    $scope.getMessage();
});
{% endhighlight %}

运行更新后的例子，两秒之后，我们就能够看到视图更新了。我们所做的唯一的改变就是将代码放进了`$scope.$apply()`中。这样，就能自动的去调用`$rootScope.$digest()`，然后，监听器就会启动，视图就会更新了。

> 注意：顺便说一句，当需要延时的时候，尽可能的使用`$timout`，这样，就不用人为的去调用`$appply()`了。

并且，注意看上面的代码，我们也可以在调用`$apply()`的时候不传参数。就像下面这样：

{% highlight javascript %}
$scope.getMessage = function() {
	setTimeout(function() {
		$scope.message = 'Fetched after two seconds';
	    console.log('message:' + $scope.message);
	    $scope.$apply(); // 这儿就触发了$digest循环
	}, 2000);
};
{% endhighlight %}

上面的代码调用`$apply()`的时候没有传入参数，但是也起作用了。记住，在调用`$apply()`的时候，应该总是要传入函数参数。因为，当你为`$apply()`传入函数的时候，这个函数在调用的时候是包含在`try..catch`中的，并且，任何发生的异常都能够被`$exceptionHandler`服务所接收。

### `$digest`循环要执行多少次呢

当一个`$digest`循环运行的时候，监听器就会被执行以查看数据模型是否发生改变。如果改变了，对应的监听函数就会被执行。这就导致了一个很重要的问题，假如一个监听函数自己改变了数据模型，AngularJS怎么知道呢？

答案就是，`$digest`循环并不只是运行一次。在当前循环的结束之后，他会再次启动来检查是否有数据发生改变。这被叫做脏检查，用来对那些可能被监听函数所更改的数据模型进行监测。所以，`$digest`循环会一直保持循环直到再也没有数据模型发生改变，或者，到达最大的循环次数(10次)。保持幂等以及最小化监听函数里的数据模型的更改总是比较好的。

> 注意：`$digest`至少会循环两次即使监听函数没有更改任何数据模型。正如上文讨论的那样，他会多运行一次以确保没有数据发生改变。

### 总结

我希望这篇文章能够清楚的描述`$digest`和`$apply`是什么。记住最重要的就是AngularJS是否能够监测到你的更改。如果不能，那么，你就得必须人为调用`$apply()`。
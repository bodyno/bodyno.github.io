---
layout: post
title:  "AngularJS：factory，service与provider的区别"
date:   2016-04-30
categories: Angular Front-End
---

翻译自[http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/](http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/)

当你开始使用Angular的时候，你会发现，你总是会让你的控制器和作用域充满各种不必要的逻辑。你应该早点意识到一个控制器应该是很简洁精炼的；同时大多数的商业逻辑和一些重复性的数据都应该要存储到服务中。一天我在Stack Overflow上看到一些问题说是考虑将重复性的数据放在控制器里，但是，这不是这不是一个控制器应该有的目的。如果为了内存需要，控制器就应该在需要他们的时候实例化，在不需要的时候就取消掉。因此，Angular在你每次切换路由的时候，就会清理当前的控制器。但是呢，服务为我们提供了一种长期存储应用数据的方式，同时，也可以在不同的控制器之间统一的使用服务。

Angular为我们提供了三种创建服务的方式：

**1、Factory**

**2、Service**

**3、Provider**

#### 先简单介绍一下

**一、**当使用`factory`来创建服务的时候，相当于新创建了一个对象，然后在这个对象上新添属性，最后返回这个对象。当把这个服务注入控制器的时候，控制器就可以访问在那个对象上的属性了。

{% highlight javascript %}
app.factory('MyFactory', function () {
        var _artist = '',
            service = {};

        service.getArtist = function () {
            return _artist;
        };

        return service;
    })
    .controller('myFactoryCtrl', [
        '$scope', 'MyFactory',
        function ( $scope, MyFactory ) {
            $scope.artist = MyFactory.getArtist();
        }]);
{% endhighlight %}

**二、**当使用`service`创建服务的时候，相当于使用`new`关键词进行了实例化。因此，你只需要在`this`上添加属性和方法，然后，服务就会自动的返回`this`。当把这个服务注入控制器的时候，控制器就可以访问在那个对象上的属性了。

{% highlight javascript %}
app.service('MyService', function () {
        var _artist = '';

        this.getArtist = function () {
            return _artist;
        };
    })
    .controller('myServiceCtrl', [
        '$scope', 'MyService',
        function ( $scope, MyService ) {
            $scope.artist = MyService.getArtist();
        }]);
{% endhighlight %}

**三、**`provider`是唯一一种可以创建用来注入到`config()`函数的服务的方式。想在你的服务启动之前，进行一些模块化的配置的话，就使用`provider`。

{% highlight javascript %}
app.provider('MyProvider', function () {

        // 只有直接添加在this上的属性才能被config函数访问
        this._artist = '';
        this.thingFromConfig = '';

        // 只有$get函数返回的属性才能被控制器访问
        this.$get = function () {
            var that = this;

            return {
                getArtist: function () {
                    return that._artist;
                },
                thingFromConfig: that.thingFromConfig
            };
        };
    })
    .config(['MyProvider', function ( MyProvider ) {
        MyProvider.thingFormConfig = 'this is set in config()';
    }])
    .controller('myProviderCtrl', [
        '$scope', 'MyProvider',
        function ( $scope, MyProvider ) {
            $scope.artist = MyProvider.getArtist();
        }]);
{% endhighlight %}

---

#### 下面我们来详细说明

为了详细的说明这三种方式的不同之处，我们分别使用这三种方式来创建同一个服务。这个服务将会用到iTunes API以及promise的`$q`。

**使用`factory`**

要创建和配置服务，最普通的做法就是使用`factory`。就像上面简单说明的那样，这里也没有太多要说明的地方，就是创建一个对象，然后为他添加属性和方法，最后返回这个对象。当把这个服务注入控制器的时候，控制器就可以访问在那个对象上的属性了。一个很普通的例子就像下面那样。

首先我们创建一个对象，然后返回这个对象。

{% highlight javascript %}
app.factory('MyFactory', function () {
    var service = {};

    return service;
});
{% endhighlight %}

现在，我们添加到`service`上的任何属性，只要将`MyFactory`注入到控制器，控制器就都可以访问了。

现在，我们添加一些私有属性到回调函数里，虽然不能从控制器里直接访问这些变量，但是最终我们会提供一些`getter`和`setter`方法到`service`上以便于我们在需要的时候修改这些属性。

{% highlight javascript %}
app.factory('MyFactory', [
    '$http', '$q', function ( $http, $q ) {
        var service = {},
            baseUrl = 'https://itunes.apple.com/search?term=',
            _artist = '',
            _finalUrl = '';

        function makeUrl() {
            _artist = _artist.split(' ').join('+');
            _finalUrl = baseUrl + _artist + '&callback=JSON_CALLBACK';
            return _finalUrl;
        }

        return service;
    }]);
{% endhighlight %}

你应该注意到了，我们没有把这些属性和方法添加到`service`对象上去。我们现在只是先简单的创建出来，以便于待会儿使用或者修改。

- `baseUrl`是iTunes API需要的基本URL
- `_artist`是我们需要查找的艺术家
- `_finalUrl`是最终向iTunes发送请求的URL
- `makeUrl`是一个用来创建返回我们最终的URL的函数

既然我们的辅助变量和函数都创建好了，那么，就往`service`添加一些属性吧。我们在`service`上添加的任何属性，只要服务注入了控制器中，那么，控制器就可以访问这些属性。

我们要创建一个`setArtist()`和`getArtist()`函数来设置以及取得艺术家的值。同时，也要创建一个用于向iTunes发送请求的函数。这个函数会返回一个promise对象，当有数据从iTunes返回的时候，这个promise对象就会执行。如果你对Angular的promise对象还不是很了解的话，推荐你去深入了解一下。

- `setArtist()`接受一个参数并且允许用来设置艺术家的值
- `getArtist()`返回艺术家的值
- `callITunes()`首先会调用`makeUrl()`函数来创建我们需要使用`$http`进行请求的URL，然后使用我们最终的URL来发送请求，创建一个promise对象。由于`$http`返回了promise对象，我们就可以在请求之后调用`.success`和`.error`了。然后我们处理从iTunes返回的数据或者驳回，并返回一个错误消息，比如`There was an error`。

{% highlight javascript %}
app.factory('MyFactory', [
    '$http', '$q', function ( $http, $q ) {
        var service = {},
            baseUrl = 'https://itunes.apple.com/search?term=',
            _artist = '',
            _finalUrl = '';

        function makeUrl() {
            _artist = _artist.split(' ').join('+');
            _finalUrl = baseUrl + _artist + '&callback=JSON_CALLBACK';
            return _finalUrl;
        }

        service.setArtist = function ( artist ) {
            _artist = artist;
        };

        service.getArtist = function () {
            return _artist;
        };

        service.callITunes = function () {
            var deferred = $q.defer();
            _finalUrl = makeUrl();

            $http({
                method: 'JSONP',
                url: _finalUrl
            }).success(function ( data ) {
                deferred.resolve(data);
            }).error(function ( error ) {
                deferred.reject(error);
            });

            return deferred.promise;
        };

        return service;
    }]);
{% endhighlight %}

现在，我们的服务就完成了，我们可以将这个服务注入到任何的控制器了，并且，可以使用我们添加到`service`上的那些方法了(`getArtist`,  `setArtise`, `callITunes`)。

{% highlight javascript %}
app.controller('myFactoryCtrl', [
    '$scope', 'MyFactory', function ( $scope, MyFactory ) {
        $scope.data = {};
        $scope.updateArtist = function () {
            MyFactory.setArtist($scope.data.artist);
        };

        $scope.submitArtist = function () {
            MyFactory.callITunes().then(function ( data ) {
                $scope.data.artistData = data;
            }, function ( error ) {
                alert(error);
            });
        };
    }]);
{% endhighlight %}

在上面的控制器中我们注入了`MyFactory`服务，然后，将从服务里来的数据设置到`$scope`的属性上。上面的代码中最难的地方应该就是你从来没有使用过promise。由于`callITunes()`返回了一个promise对象，所以一旦有数据从iTunes返回，promise执行的时候，我们就可以使用`.then()`方法来设置`$scope.data.artistData`的值了。你会注意到，我们的控制器非常简洁，我们所有的逻辑和重复性数据都写在了服务里面。

**使用`service`**

也许在使用`service`创建服务时，我们需要知道的最重要的一件事就是他是使用`new`关键字进行实例化的。如果你是Javascript大师，你应该知道从代码的本质来思考。对于那些不了解Javascript背景的或者并不熟悉`new`实际做了什么的程序员，我们需要复习一下Javascript的基础知识，以便于最终帮助我们理解`service`的本质。

为了真正的看到当我们使用`new`来调用函数的时候发生了什么，我们来创建一个函数，并且使用`new`来调用他，然后，我们再看看在解释器发现`new`的时候，他会做什么。最终结果肯定是一样的。

首先创建我们的构造函数：

{% highlight javascript %}
function Person( name, age ) {
    this.name = name;
    this.age = age;
}
{% endhighlight %}

这是一个典型的构造函数。现在，无论我们什么时候使用`new`来调用这个函数，`this`都会被绑定到新创建的那个对象上。

现在我们再在`Person`的原型上创建一个方法，以便于每一个实例都可以访问到。

{% highlight javascript %}
Person.prototype.sayName = function () {
    alert('My name is: ' + this.name);
};
{% endhighlight %}

现在，由于我们在`Person`对象的原型上创建了`sayName`函数，所以，`Person`的每一个实例都可以调用到这个方法。

既然我们已经有了构造函数和原型方法，那么，就来真正的创建一个`Person`的实例并且调用`sayName`函数：

{% highlight javascript %}
var tyler = new Person('Tyler', 23);
tyler.sayName();
{% endhighlight %}

所以，最终，所有的代码合起来就是下面这个样子：

{% highlight javascript %}
function Person( name, age ) {
    this.name = name;
    this.age = age;
}

Person.prototype.sayName = function () {
    alert('My name is: ' + this.name);
};

var tyler = new Person('Tyler', 23);
tyler.sayName();
{% endhighlight %}

现在我们来看看在使用`new`的时候到底发生了什么。首先你应该注意到的是，在我们的例子中，使用了`new`之后，我们可以使用`tyler`来调用`sayName`方法，就好像这是一个对象一样，当然，`tyler`确实是一个对象。所以，我们首先知道的就是无论我们是否能够在代码里面看见，`Person`构造函数是会返回一个对象的。第二，我们我们应该知道，`sayName`方法是在原型上的，不是直接定义在`Person`对象实例上的，所以，`Person`返回的对象必须是通过原型委托的。用更简单的例子说就是，当我们调用`tyler.sayName()`的时候，解释器就会说：“OK，我将会在刚创建的`tyler`对象上查找`sayName`函数，然后调用他。等会儿，我没有发现这个函数，只看到了`name`和`age`属性，让我再检查一下原型。哦，原来在原型上，让我来调用他”。

下面的代码就是你能够想象的在Javascript里，`new`实际做了什么。下面的代码是一个很基础的例子，我以解释器的视角来添加了一些注释：

{% highlight javascript %}
function Person( name, age ) {
    //var obj = object.create(Person.prototype);
    //this = obj;

    this.name = name;
    this.age = age;

    //return this;
}
{% endhighlight %}

现在，既然知道了`new`做了什么，那么，使用`service`来创建服务也很容易理解了。

在使用`service`创建服务时，我们需要知道的最重要的一件事就是他是使用`new`关键字进行实例化的。与上面的例子的知识相结合，你应该就能意识到你要把属性和方法添加到`this`上，并且，服务会自动返回`this`。

与我们使用`factory`创建服务的方式不同，我们不需要新创建一个对象然后再返回这个对象，因为正如我们前面所提到的那样，我们使用`new`的时候，解释器会自动创建对象，并且代理到他的原型，然后代替我们返回。

所以，在所有的开始之前，我们先创建我们的私有辅助函数，与我们之前使用`factory`创建的时候非常类似。现在我不会解释每一行的意义了，如果你有什么疑惑的话，可以看看前面的`factory`的例子。

{% highlight javascript %}
app.service('MyService', [
    '$http', '$q', function ( $http, $q ) {
        var baseUrl = 'https://itunes.apple.com/search?term=',
            _artist = '',
            _finalUrl = '';

        function makeUrl() {
            _artist = _artist.split(' ').join('+');
            _finalUrl = baseUrl + _artist + '&callback=JSON_CALLBACK';
            return _finalUrl;
        }
    }]);
{% endhighlight %}

现在，我们会把可用的方法都添加到`this`上。

{% highlight javascript %}
app.service('MyService', [
    '$http', '$q', function ( $http, $q ) {
        var baseUrl = 'https://itunes.apple.com/search?term=',
            _artist = '',
            _finalUrl = '';

        function makeUrl() {
            _artist = _artist.split(' ').join('+');
            _finalUrl = baseUrl + _artist + '&callback=JSON_CALLBACK';
            return _finalUrl;
        }

        this.setArtist = function ( artist ) {
            _artist = artist;
        };

        this.getArtist = function () {
            return _artist;
        };

        this.callITunes = function () {
            var deferred = $q.defer();
            _finalUrl = makeUrl();

            $http({
                method: 'JSONP',
                url: _finalUrl
            }).success(function ( data ) {
                deferred.resolve(data);
            }).error(function ( error ) {
                deferred.reject(error);
            });

            return deferred.promise;
        };
    }]);
{% endhighlight %}

现在，就像我们使用`factory`所创建的服务那样，注入这个服务的任何一个控制器都可以使用`setArtist`，`getArtist`和`callITunes`方法了。下面是我们的`myServiceCtrl`，几乎与`myFactoryCtrl`相同。

{% highlight javascript %}
app.controller('myServiceCtrl', [
    '$scope', 'MyService', function ( $scope, MyService ) {
        $scope.data = {};
        $scope.updateArtist = function () {
            MyService.setArtist($scope.data.artist);
        };

        $scope.submitArtist = function () {
            MyService.callITunes().then(function ( data ) {
                $scope.data.artistData = data;
            }, function ( error ) {
                alert(error);
            });
        };
    }]);
{% endhighlight %}

正如我之前提到的，一旦你理解了`new`关键词做了什么，`service`和`factory`就几乎是相同的。


**使用`provider`**

关于`provider`，要记住的最重要的一件事就是他是唯一一种可以创建用来注入到`app.config()`函数的服务的方式。

如果你需要在你的应用在别处运行之前对你的服务对象进行一部分的配置，那么，这个就显得很重要了。尽管与`service`和`provider`类似，但是我们还是会讲解一些他们的不同之处。

首先，类似的，我们设置我们的`provider`。下面的变量就是我们的私有函数。

{% highlight javascript %}
app.provider('MyProvider', function () {
    var baseUrl = 'https://itunes.apple.com/search?term=',
        _artist = '',
        _finalUrl = '';

    // 从config函数里设置这个属性
    this.thingFromConfig = '';

    function makeUrl() {
        _artist = _artist.split(' ').join('+');
        _finalUrl = baseUrl + _artist + '&callback=JSON_CALLBACK';
        return _finalUrl;
    }
});
{% endhighlight %}

> 再说明一次，如果对上面的代码逻辑有疑问的话，可以参考之前的列子。

你可以认为`provider`有三个部分，第一部分是私有变量和私有函数，这些变量和函数会在以后被修改。第二部分是在`app.config`函数里可以访问的变量和函数，所以，他们可以在在其他地方使用之前被修改。注意，这些变量和函数一定要添加到`this`上面才行。在我们的例子中，`app.config()`函数能够修改的只有`thingFromConfig`。第三部分是在控制器里可以访问的变量和函数。

当使用 `provider`创建服务的时候，唯一可以让控制器访问的属性和方法是在`$get()`函数里返回的属性和方法。下面的代码将`$get`添加到了`this`上面，最终这个函数会被返回。

现在，`$get()`函数会返回我们需要在控制器里访问的函数和变量。下面是代码例子：

{% highlight javascript %}
this.$get = function ( $http, $q ) {
            return {
                setArtist: function ( artist ) {
                    _artist = artist;
                },
                getArtist: function () {
                    return _artist;
                },
                callITunes: function () {
                    var deferred = $q.defer();
                    _finalUrl = makeUrl();

                    $http({
                        method: 'JSONP',
                        url: _finalUrl
                    }).success(function ( data ) {
                        deferred.resolve(data);
                    }).error(function ( error ) {
                        deferred.reject(error);
                    });

                    return deferred.promise;
                },
                thingOnConfig: this.thingFromConfig
            };
        };
{% endhighlight %}

现在，完整的`provider`就是这个样子：

{% highlight javascript %}
app.provider('MyProvider', [
    '$http', '$q', function ( $http, $q ) {
        var baseUrl = 'https://itunes.apple.com/search?term=',
            _artist = '',
            _finalUrl = '';

        this.thingFromConfig = '';

        this.$get = function ( $http, $q ) {
            return {
                setArtist: function ( artist ) {
                    _artist = artist;
                },
                getArtist: function () {
                    return _artist;
                },
                callITunes: function () {
                    var deferred = $q.defer();
                    _finalUrl = makeUrl();

                    $http({
                        method: 'JSONP',
                        url: _finalUrl
                    }).success(function ( data ) {
                        deferred.resolve(data);
                    }).error(function ( error ) {
                        deferred.reject(error);
                    });

                    return deferred.promise;
                },
                thingOnConfig: this.thingFromConfig
            };
        };

        function makeUrl() {
            _artist = _artist.split(' ').join('+');
            _finalUrl = baseUrl + _artist + '&callback=JSON_CALLBACK';
            return _finalUrl;
        }
    }]);
{% endhighlight %}

现在，与之前的`service`和`factory`类似，只要我们把`MyProvider`注入到控制器里面，对应的方法就可以使用了。下面是`myProviderCtrl`。

{% highlight javascript %}
app.controller('myProviderCtrl', [
    '$scope', 'MyProvider', function ( $scope, MyProvider ) {
        $scope.data = {};
        $scope.updateArtist = function () {
            MyProvider.setArtist($scope.data.artist);
        };

        $scope.submitArtist = function () {
            MyProvider.callITunes().then(function ( data ) {
                $scope.data.artistData = data;
            }, function ( error ) {
                alert(error);
            });
        };

        $scope.data.thingFromConfig = MyProvider.thingOnConfig;
    }]);
{% endhighlight %}

正如之前提到的，使用`provider`来创建服务的目的就是为了能够通过`app.config()`函数修改一些变量来传递到最终的项目中。我们来看个例子：

{% highlight javascript %}
app.config(['MyProviderProvider', function ( MyProviderProvider ) {
    MyProviderProvider.thingFromConfig = 'This sentence was set in app.config. Providers are the only service that can be passed into app.config. Check out the code to see how it works.';
}]);
{% endhighlight %}

现在，你就能看到，在`provider`里，`thingFromConfig`是空字符串，但是，当我们在DOM里显示的时候，他就会是我们上面所设置的字符串了。

谢谢你的阅读，希望能够帮助你分辨这三者的不同之处。

> 要查看完整的代码例子，欢迎fork我的项目：https://github.com/tylermcginnis33/AngularServices 或者查看[我在Stack Overflow的问题回答](http://stackoverflow.com/a/23683176/1867084)

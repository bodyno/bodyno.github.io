---
layout: post
title:  "Ruby Sass和LibSass的区别"
date:   2016-05-07
categories: Front-End
---

翻译自[http://sassbreak.com/ruby-sass-libsass-differences/](http://sassbreak.com/ruby-sass-libsass-differences/)

当你习惯了使用Sass来工作之后，你会听说一种新的Sass类型叫做LibSass。然后，你马上就会想：“LibSass到底是什么？我需要从Ruby Sass转换到LibSass吗？噢不不，我应该只是在跟风而已”。

别担心，你没有跟风。如你所学到的，Ruby Sass和LibSass是类似的。主要的区别在于它们的实现，编译速度和功能支持。在本文中，我将会联同它们的优缺点一起来叙述这两者的区别。

### Ruby Sass

最原始的Sass是使用Ruby编写的。如果你刚开始使用Sass，那么你会接触到基于Ruby的Sass。只要你安装了Ruby和Sass，运行编译器，就可以使用Sass提供的令人惊叹的功能了。

但是，如果开发环境里没有Ruby，那么Ruby Sass也就没用了。并且，依据项目的大小，Ruby Sass引擎可能会花大部分时间来将Sass编译为CSS。在大型项目这种现象很明显。项目越大，编译速度越慢。

所以，如果在开发环境中没有Ruby该怎么办呢？或者说，怎么处理大型项目中编译速度慢的问题呢？

### 进入Ruby Sass的世界

LibSass是原始Sass引擎的一个C/C++接口。使用它编译Sass的时候并不依赖于Ruby，所以，可以使用其他的语言来实现LibSass。

LibSass本身不做任何事情，它只是一个库。要使用LibSass来工作，你需要一个包装器(或者说一个实现器)来包装LibSass并编译你的样式表。最普通的LibSass的实现器是SassC，也是最先开发出来的；然后就是node-sass，grunt-sass，甚至还有Ruby的实现器。

例如，一个工程在运行node-sass，那么，LibSass就是核心库，而node-sass就是一个允许在Node.js里编译的包装器。

### 为什么要使用LibSass？

使用LibSass的话，你就不需要依赖于Ruby环境了。并且，有很多可用的包装器，可以很容易的将LibSass与其他任何语言整合起来。

LibSass最大的优点就是编译速度。LibSass运行和编译样式的速度明显快于Ruby Sass，比Ruby Sass快4000%！可以查看这篇文章来比较Sass，LibSass和其他CSS预处理器的编译速度：[compile time comparison](http://www.solitr.com/blog/2014/01/css-preprocessor-benchmark/)。

### 与Ruby Sass的兼容

尽管LibSass还不算完善，但是，许多开发者在犹豫是否要将它们的代码迁移到LibSass。因为在新功能出现的时候，LibSass要落后Ruby Sass一些。

例如，Sass 3.4中的新功能`@at-root, @error`在LibSass上还不受支持。要想知道大部分的Sass，LibSass和旧版本的Sass引擎的兼容性，可以查看这个：[sass-compatibility.github.io](http://sass-compatibility.github.io)。

**LibSass的未来是有希望的**

好休息就是LibSass马上就会赶上Ruby Sass了。即将到来的LibSass 3.2版本将会支持`@at-root, @error, @media, @keyframes`等更多的指令。

事实上，在LibSass添加上Ruby Sass已有的功能之前，Ruby Sass是不会发布任何新功能的。一旦发布新功能，这两者的功能就是一样的了。所以，很快，我们就可以在LibSass编译速度的基础上的拥有Ruby Sass的全部功能了。

想要快速测试LibSass和Ruby 的功能，可以查看[SassMeister](http://www.sassmeister.com)。

### 最后的一点想法

一旦你设置好了LibSass之后，你会发现，使用起来与Ruby Sass一样简单，仅仅是更快了而已。如果你使用的只是Sass的核心功能，那么，你迁移到LibSass就会很容易了。

如果你和你的团队以及你的项目，都很乐于使用Ruby Sass，那么就继续使用吧。但是，根据你的项目情况，LibSass的速度可能值得暂时性的缺失一些Sass的功能。
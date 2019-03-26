---
layout: post
title: "JS闭包的底层运行机制"
date: 2019-03-26
tags: [javascript]
categories: javascript
---

我研究JavaScript闭包（closure）已经有一段时间了。我之前只是学会了如何使用它们，而没有透彻地了解它们具体是如何运作的。那么，究竟什么是闭包？

[Wikipedia](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29)给出的解释并没有太大的帮助。闭包是什么时候被创建的，什么时候被销毁的？具体的实现又是怎么样的？

````
"use strict";  
  
var myClosure = (function outerFunction() {  
  
 var hidden = 1;  
  
 return {  
 inc: function innerFunction() {  
 return hidden++;  
 }  
 };  
  
}());  
  
myClosure.inc();  // 返回 1  
myClosure.inc();  // 返回 2  
myClosure.inc();  // 返回 3  
````
  
// 相信对JS熟悉的朋友都能很快理解这段代码  
// 那么在这段代码运行的背后究竟发生了怎样的事情呢？  

现在，我终于知道了答案，我感到很兴奋并且决定向大家解释这个答案。至少，我一定是不会忘记这个答案的。

> Tell me and I forget. Teach me and I remember. Involve me and I learn.  
> © Benjamin Franklin

并且，在我阅读与闭包相关的现存的资料时，我很努力地尝试着去在脑海中想想每个事物之间的联系：对象之间是如何引用的，对象之间的继承关系是什么，等等。我找不到关于这些负责关系的很好的图表，于是我决定自己画一些。

我将假设读者对JavaScript已经比较熟悉了，知道什么是全局对象，知道函数在JavaScript当中是“first-class objects”，等等。

### [](#作用域链（Scope-Chain） "作用域链（Scope Chain）")作用域链（Scope Chain）

当JavaScript在运行的时候，它需要一些空间让它来存储本地变量（local variables）。我们将这些空间称为作用域对象（Scope object），有时候也称作`LexicalEnvironment`。例如，当你调用函数时，函数定义了一些本地变量，这些变量就被存储在一个作用域对象中。你可以将作用域函数想象成一个普通的JavaScript对象，但是有一个很大的区别就是你不能够直接在JavaScript当中直接获取这个对象。你只可以修改这个对象的属性，但是你不能够获取这个对象的引用。

作用域对象的概念使得JavaScript和C、C++非常不同。在C、C++中，本地变量被保存在栈（stack）中。**在JavaScript中，作用域对象是在堆中被创建的（至少表现出来的行为是这样的），所以在函数返回后它们也还是能够被访问到而不被销毁。**

正如你做想的，作用域对象是可以有父作用域对象（parent scope object）的。当代码试图访问一个变量的时候，解释器将在当前的作用域对象中查找这个属性。如果这个属性不存在，那么解释器就会在父作用域对象中查找这个属性。就这样，一直向父作用域对象查找，直到找到该属性或者再也没有父作用域对象。我们将这个查找变量的过程中所经过的作用域对象乘坐作用域链（Scope chain）。

在作用域链中查找变量的过程和原型继承（prototypal inheritance）有着非常相似之处。但是，非常不一样的地方在于，当你在原型链（prototype chain）中找不到一个属性的时候，并不会引发一个错误，而是会得到`undefined`。但是如果你试图访问一个作用域链中不存在的属性的话，你就会得到一个`ReferenceError`。

在作用域链的最顶层的元素就是全局对象（Global Object）了。运行在全局环境的JavaScript代码中，作用域链始终只含有一个元素，那就是全局对象。所以，当你在全局环境中定义变量的时候，它们就会被定义到全局对象中。当函数被调用的时候，作用域链就会包含多个作用域对象。

### [](#全局环境中运行的代码 "全局环境中运行的代码")全局环境中运行的代码

好了，理论就说到这里。接下来我们来从实际的代码入手。

````
// my\_script.js  
"use strict";  
  
var foo = 1;  
var bar = 2;  
````

我们在全局环境中创建了两个变量。正如我刚才所说，此时的作用域对象就是全局对象。

![](/assets/closure/js_closure_1.png "在全局环境中创建两个变量")

在上面的代码中，我们有一个执行的上下文（**myscript.js**自身的代码），以及它所引用的作用域对象。全局对象里面还含有很多不同的属性，在这里我们就忽略掉了。

### [](#没有被嵌套的函数（Non-nested-functions） "没有被嵌套的函数（Non-nested functions）")没有被嵌套的函数（Non-nested functions）

接下来，我们看这段代码

````
"use strict";  
var foo = 1;  
var bar = 2;  
  
function myFunc() {  
 //-- define local-to-function variables  
 var a = 1;  
 var b = 2;  
 var foo = 3;  
  
 console.log("inside myFunc");  
}  
  
console.log("outside");  
  
//-- and then, call it:  
myFunc();  
````

当`myFunc`被定义的时候，`myFunc`的标识符（identifier）就被加到了当前的作用域对象中（在这里就是全局对象），并且这个标识符所引用的是一个函数对象（function object）。函数对象中所包含的是函数的源代码以及其他的属性。其中一个我们所关心的属性就是内部属性`[[scope]]`。`[[scope]]`所指向的就是当前的作用域对象。也就是指的就是函数的标识符被创建的时候，我们所能够直接访问的那个作用域对象（在这里就是全局对象）。

> “直接访问”的意思就是，在当前作用域链中，该作用域对象处于最底层，没有子作用域对象。

所以，在`console.log("outside")`被运行之前，对象之间的关系是如下图所示。

![](/assets/closure/js_closure_2.png "对象之间的关系")

温习一下。`myFunc`所引用的函数对象其本身不仅仅含有函数的代码，并且还含有指向其**被创建的时候的作用域对象**。这一点**非常重要！**

当`myFunc`函数被调用的时候，一个新的作用域对象被创建了。新的作用域对象中包含`myFunc`函数所定义的本地变量，以及其参数（arguments）。这个新的作用域对象的父作用域对象就是在运行`myFunc`时我们所能直接访问的那个作用域对象。

所以，当`myFunc`被执行的时候，对象之间的关系如下图所示。

![](/assets/closure/js_closure_3.png "对象之间的关系（函数执行后）")

现在我们就拥有了一个作用域链。当我们试图在`myFunc`当中访问某些变量的时候，JavaScript会先在其能直接访问的作用域对象（这里就是`myFunc() scope`）当中查找这个属性。如果找不到，那么就在它的父作用域对象当中查找（在这里就是`Global Object`）。如果一直往上找，找到没有父作用域对象为止还没有找到的话，那么就会抛出一个`ReferenceError`。

例如，如果我们在`myFunc`中要访问`a`这个变量，那么在`myFunc scope`当中就可以找到它，得到值为`1`。

如果我们尝试访问`foo`，我们就会在`myFunc() scope`中得到`3`。只有在`myFunc() scope`里面找不到`foo`的时候，JavaScript才会往`Global Object`去查找。所以，这里我们不会访问到`Global Object`里面的`foo`。

如果我们尝试访问`bar`，我们在`myFunc() scope`当中找不到它，于是就会在`Global Object`当中查找，因此查找到2。

很重要的是，只要这些作用域对象依然被引用，它们就不会被垃圾回收器（garbage collector）销毁，我们就一直能访问它们。当然，**当引用一个作用域对象的最后一个引用被解除的时候，并不代表垃圾回收器会立刻回收它，只是它现在可以被回收了**。

所以，当`myFunc()`返回的时候，再也没有人引用`myFunc() scope`了。当垃圾回收结束后，对象之间的关系变成回了调用前的关系。

![](/assets/closure/js_closure_2.png "对象之间的关系恢复")

接下来，为了图表直观起见，我将不再将函数对象画出来。但是，请永远记着，函数对象里面的`[[scope]]`属性，保存着该函数被定义的时候所能够直接访问的作用域对象。

### [](#嵌套的函数（Nested-functions） "嵌套的函数（Nested functions）")嵌套的函数（Nested functions）

正如前面所说，当一个函数返回后，没有其他对象会保存对其的引用。所以，它就可能被垃圾回收器回收。但是如果我们在函数当中定义嵌套的函数并且返回，被调用函数的一方所存储呢？（如下面的代码）
 
````
function myFunc() {  
 return innerFunc() {  
 // ...  
 }  
}  
  
var innerFunc = myFunc();  
````

你已经知道的是，函数对象中总是有一个`[[scope]]`属性，保存着该函数被定义的时候所能够直接访问的作用域对象。所以，当我们在定义嵌套的函数的时候，这个嵌套的函数的`[[scope]]`就会引用外围函数（Outer function）的当前作用域对象。

如果我们将这个嵌套函数返回，并被另外一个地方的标识符所引用的话，那么这个嵌套函数及其`[[scope]]`所引用的作用域对象就不会被垃圾回收所销毁。

````
"use strict";  
  
function createCounter(initial) {  
 var counter = initial;  
  
 function increment(value) {  
 counter += value;  
 }  
  
 function get() {  
 return counter;  
 }  
  
 return {  
 increment: increment,  
 get: get  
 };  
}  
  
var myCounter = createCounter(100);  
  
console.log(myCounter.get());   // 返回 100  
myCounter.increment(5);  
console.log(myCounter.get());   // 返回 105  
````

当我们调用`createCounter(100)`的那一瞬间，对象之间的关系如下图

![](/assets/closure/js_closure_4.png "调用createCounter(100)时对象间的关系")

注意`increment`和`get`函数都存有指向`createCounter(100) scope`的引用。如果`createCounter(100)`没有任何返回值，那么`createCounter(100) scope`不再被引用，于是就可以被垃圾回收。但是因为`createCounter(100)`实际上是有返回值的，并且返回值被存储在了`myCounter`中，所以对象之间的引用关系变成了如下图所示

![](/assets/closure/js_closure_5.png "调用createCounter(100)后对象间的关系")

所以，`createCounter(100)`虽然已经返回了，但是它的作用域对象依然存在，可以**且仅只能**被嵌套的函数（`increment`和`get`）所访问。

让我们试着运行`myCounter.get()`。刚才说过，函数被调用的时候会创建一个新的作用域对象，并且该作用域对象的父作用域对象会是当前可以直接访问的作用域对象。所以，当`myCounter.get()`被调用时的一瞬间，对象之间的关系如下。

![](/assets/closure/js_closure_5.png "调用myCounter.get()对象间的关系")

在`myCounter.get()`运行的过程中，作用域链最底层的对象就是`get() scope`，这是一个空对象。所以，当`myCounter.get()`访问`counter`变量时，JavaScript在`get() scope`中找不到这个属性，于是就向上到`createCounter(100) scope`当中查找。然后，`myCounter.get()`将这个值返回。

调用`myCounter.increment(5)`的时候，事情变得更有趣了，因为这个时候函数调用的时候传入了参数。

![](/assets/closure/js_closure_6_inc.png "调用myCounter.increment(5)对象间的关系")

正如你所见，`increment(5)`的调用创建了一个新的作用域对象，并且其中含有传入的参数`value`。当这个函数尝试访问`value`的时候，JavaScript立刻就能在当前的作用域对象找到它。然而，这个函数试图访问`counter`的时候，JavaScript无法在当前的作用域对象找到它，于是就会在其父作用域`createCounter(100) scope`中查找。

我们可以注意到，在`createCounter`函数之外，除了被返回的`get`和`increment`两个方法，没有其他的地方可以访问到`value`这个变量了。**这就是用闭包实现“私有变量”的方法**。

我们注意到`initial`变量也被存储在`createCounter()`所创建的作用域对象中，尽管它没有被用到。所以，我们实际上可以去掉`var counter = initial;`，将`initial`改名为`counter`。但是为了代码的可读性起见，我们保留原有的代码不做变化。

需要注意的是作用域链是不会被复制的。每次函数调用只会往作用域链下面新增一个作用域对象。所以，如果在函数调用的过程当中对作用域链中的任何一个作用域对象的变量进行修改的话，那么同时作用域链中也拥有该作用域对象的函数对象也是能够访问到这个变化后的变量的。

这也就是为什么下面这个大家都很熟悉的例子会不能产出我们想要的结果。

````
"use strict";  
  
var elems = document.getElementsByClassName("myClass"), i;  
  
for (i = 0; i < elems.length; i++) {  
 elems\[i\].addEventListener("click", function () {  
 this.innerHTML = i;  
 });  
}  
````

在上面的循环中创建了多个函数对象，所有的函数对象的`[[scope]]`都保存着对当前作用域对象的引用。而变量`i`正好就在当前作用域链中，所以循环每次对`i`的修改，对于每个函数对象都是能够看到的。

### [](#“看起来一样的”函数，不一样的作用域对象 "“看起来一样的”函数，不一样的作用域对象")“看起来一样的”函数，不一样的作用域对象

现在我们来看一个更有趣的例子。

````
"use strict";  
  
function createCounter(initial) {  
 // ...  
}  
  
var myCounter1 = createCounter(100);  
var myCounter2 = createCounter(200);  
````

当`myCounter1`和`myCounter2`被创建后，对象之间的关系为

![](/assets/closure/js_closure_7.png "myCounter1和myCounter2被创建后，对象之间的关系")

在上面的例子中，`myCounter1.increment`和`myCounter2.increment`的函数对象拥有着一样的代码以及一样的属性值（`name`，`length`等等），但是它们的`[[scope]]`指向的是**不一样的作用域对象**。

这才有了下面的结果

````
var a, b;  
a = myCounter1.get();   // a 等于 100  
b = myCounter2.get();   // b 等于 200  
  
myCounter1.increment(1);  
myCounter1.increment(2);  
  
myCounter2.increment(5);  
  
a = myCounter1.get();   // a 等于 103  
b = myCounter2.get();   // b 等于 205  
````

### [](#作用域链和this "作用域链和this")作用域链和`this`

`this`的值不会被保存在作用域链中，`this`的值取决于函数被调用的时候的情景。

### [](#总结 "总结")总结

让我们来回想我们在本文开头提到的一些问题。

*   什么是闭包？闭包就是同时含有对函数对象以及作用域对象引用的对象。实际上，所有JavaScript对象都是闭包。
*   闭包是什么时候被创建的？因为所有JavaScript对象都是闭包，因此，当你定义一个函数的时候，你就定义了一个闭包。
*   闭包是什么时候被销毁的？当它不被任何其他的对象引用的时候。

### [](#专有名词翻译表 "专有名词翻译表")专有名词翻译表

本文采用下面的专有名词翻译表，如有更好的翻译请告知，尤其是加`*`的翻译

*   \*全局环境中运行的代码：top-level code
*   参数：arguments
*   作用域对象：Scope object
*   作用域链：Scope Chain
*   栈：stack
*   原型继承：prototypal inheritance
*   原型链：prototype chain
*   全局对象：Global Object
*   标识符：identifier
*   垃圾回收器：garbage collector

### [](#著作权声明 "著作权声明")著作权声明

本文经授权翻译自[How do JavaScript closures work under the hood](http://dmitryfrank.com/articles/js_closures?utm_source=javascriptweekly&utm_medium=email)。

译者对原文进行了描述上的一些修改。但在没有特殊注明的情况下，译者表述的意思和原文保持一致。
---
layout: post
title: "TypeScript类型元编程"
date: 2022-02-18
tags: [meta programing typescript]
categories: typescript
---

### TypeScript 类型体操

已经2022年了，相信大家已经在项目中都用到了 TypeScript，

这里基础的概念我就不介绍了，重点给大家讲一下 TS 的一些有趣玩法，下面用 TS 简称 TypeScript 。

TS 能帮我们编写更好的代码，有类型的检查，一些空引用，未定义的变量这些都可以很好的检查出来，在开发环节就解决掉。

进入正题，什么是元编程

Meta 是吧。是 FaceBook 吗？是元宇宙吗?大家知道是什么吗？

元数据，大家都知道，就是定义数据的数据，比如 SQL 的表结构，它就是用来定义 SQL 数据的数据，

同样，元编程就是通过编程来编程，通俗理解就是生成代码。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f90d127dea9d402fb421f65c8bb44cda~tplv-k3u1fbpfcp-watermark.image?)

构成元编程的要素，一个就是元，也就是用代码来生成代码。什么叫编程呢？CSS 我们一般不说它可以用来编程，为什么呢？因为它这个语言不是图灵完备的语言。所以如果我们想要元编程，就是用代码生成代码，一定要满足图灵完备性。

图灵完备的语言，什么是图灵完备，就是在可计算性理论里，如果一系列操作数据的规则可以用来模拟**图灵机**，我们就称它是图灵完备的。

不好理解是吧，不好理解就对了，讲人话就是说：这门语言是完备的，什么叫完备的，也就是有分支、循环、跳转这些功能。有了这些功能，就可以编程，解决问题了。

图灵完备的语言大致分为三大类：
一种是顺序 + goto
一种是类C，包括我们的 JS ，有顺序、分支、循环
还有一种是函数式，分支 + 递归

当然，TS 也是一门图灵完备的语言，属于是函数式这一种，它的分支、循环、跳转这些语法和我们熟悉的 JS 语法是有一些不一样的。

比如：

**分支**：条件类型 也就是 if else：

TS 是使用 extends 语法

``` typescript
类型表达式1 extends 类型表达式2 ？ 类型表达式 : 类型表达式
```

``` typescript
type C = 'a' extends 'a' | 'b' ? true : false
/**
type C = true
*/
```

循环：在 TS 使用的是泛型递归

通过泛型经过一系列类型计算，最后产生出新的类型，这就相当于 JS 中我们使用的函数，可以传递参数，根据传入参数的不同，产出的结果也不一样。TS 没有 for 循环，我们就用递归，自己调用自己实现循环。

这里有个注意的点，TS 的递归是有限制的，次数不能太多，否则会计算不出来。不过这可以通过一些优化的手段来解决，比如建表之类的。用递归有一个关键的点就是退出递归的条件，这个一定不能忘记写，否则就是无限循环了，会导致爆栈。

``` typescript
// infer 可以匹配这个位置的类型，将该类型赋值给一个临时的类型变量，然后在后续的 If 中使用
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

``` typescript
type T = Flatten<string[]>;
/* T = string */

// 如果要把 string[][] 类型给打平呢？
type T = Flatten<string[][]>;
```

``` typescript
type Flatten<Type> = Type extends Array<infer Item> ? Flatten<Item> : Type;
```

``` typescript
type T = Flatten<string[][]>;
/* T = string */
```

这里要注意的是：一开始写递归会有点不习惯，思维上面要有一点转化，熟能生巧，可以多做一些 TS 类型体操的练习题，[type-challenges](https://github.com/type-challenges/type-challenges)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f130f0ab99ac40619f7b82e73a6bdec6~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e10b11722a5249239c1a7b3d4ed7c77c~tplv-k3u1fbpfcp-watermark.image?)

可以看到，题目已经很多了，平时无聊的时候可以来练练手，当然这和刷 LeetCode 算法题不一样，卡住你的主要是一些 TS 语法不熟悉，卡住了就看答案就好了，快速过一下，增加语法的熟练度。


### 实现加法

有了分支，循环，这些语法，我们就可以实现加法了，这里我在线给大家演示一下，方便大家理解。


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f45185e34da94b819f1bdecbe22128c3~tplv-k3u1fbpfcp-watermark.image?)

这样写 TS 肯定是解析不了的，之所以叫类型体操，肯定没有这么简单的是吧。

``` typescript
type NumberToArray<T, Result extends any[] = []> = Result['length'] extends T 
? Result 
: NumberToArray<T, [...Result, any]>

type Add<Num1, Num2> = [...NumberToArray<Num1>, ...NumberToArray<Num2>]['length']

type Result = Add<5, 6>
/* Result = 11 */
``` 

可以看到，我们通过将参数传入泛型类型，经过一系列运算，将数字类型转化成数组类型，然后不断判断数组的长度是否满足要求，不满足要求则再往数组里加一个元素，这个元素是什么类型我们不关心，关键是数组的个数增加一个就行了，最后当数组长度满足时，我们结束递归，返回数组，再通过将两个数组合并，得到大数组的长度，即得到了相加的结果。

如果想计算更多个数的相加，我们重复使用 Add 泛型就可以了。

``` typescript
type Result = Add<1, Add<2, 3>>
/* Result = 6 */
``` 

同样的，我们可以基于数组的原理实现减法，方法很简单，通过 infer 占位匹配就可以了。

``` typescript
type NumberToArray<T, Result extends any[] = []> = Result['length'] extends T 
? Result 
: NumberToArray<T, [...Result, any]>

type Sub<Num1, Num2> = NumberToArray<Num1> extends [...NumberToArray<Num2>, ...infer Result] 
? Result['length'] 
: never

type Result =Sub<3, 1>
/* Result = 2 */
``` 

相应的，有了加法，减法，【乘法，除法】的实现就可以依葫芦画瓢。这里我就不一一实现了，有兴趣的小伙伴可以自己去下面实现。


【加减乘除】都有了，加上我们的分支循环，我们的编程大厦的底座就有了，以后一些通用的需求就都可以用 TS 来实现了。

像 TS 的一些内置类型，我们完全可以自己实现。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f86c497480954acda3c00f25263678d6~tplv-k3u1fbpfcp-watermark.image?)

然后就有了大神们的一通骚操作：

[TypeScript 类型体操天花板，用类型运算写一个 Lisp 解释器](https://zhuanlan.zhihu.com/p/427309936)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dc2086049e54bc4a369dedcb32146bc~tplv-k3u1fbpfcp-watermark.image?)

[# 用 TypeScript 类型运算实现一个中国象棋程序](https://zhuanlan.zhihu.com/p/426966480)

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbe893500f6c47808f92483257b54c97~tplv-k3u1fbpfcp-watermark.image?)

[ts-toolbelt: TypeScript's largest type utility library](https://github.com/millsp/ts-toolbelt)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d2f01950d234e5aaffd8e9f7fd3d4fd~tplv-k3u1fbpfcp-watermark.image?)

[typetype: A programming language designed for typescript type generation](https://github.com/mistlog/typetype)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1c621354331440fab7fd83340b36fe86~tplv-k3u1fbpfcp-watermark.image?)



### TS 类型体操，你学废了吗？


![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c34bce9761534b5a8543f2e755ce886f~tplv-k3u1fbpfcp-watermark.image?)


参考资料

[1] https://www.typescriptlang.org/

[2] TypeScript 类型编程: 从基础到编译器实战：https://mp.weixin.qq.com/s/-x8iVK-hlQd3-OZDC04A5A

[3] 模式匹配-让你 ts 类型体操水平暴增的套路：https://mp.weixin.qq.com/s/wLTCyRhXX3HQjvDSm7WgEQ

[4] 来做操吧！深入 TypeScript 高级类型和类型体操 https://mp.weixin.qq.com/s/z5N3ePnhAo4liYaZLfwtog

















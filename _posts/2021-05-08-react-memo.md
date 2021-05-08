---
layout: post
title: "React Memo"
date: 2021-05-08
tags: [javascript]
categories: javascript
---

#### A

React 将组件树的render从同步（Legacy Mode） 变为可中断的异步（Concurrent Mode）花了2年

其中主要包括
Stack Reconciler -> Fiber Reconciler

Schedule

ExpirationTime -> Lane

Fiber很多出，大多数前端都听说过

#### B

effect list 是 React 源码中 commit 阶段的一个特性

React内部工作大体可以分为3个阶段：

调度更新

决定什么组件需要更新

更新组件

那么第三步如何知道要更新哪些组件呢？靠effect list。

如果将React Fiber树比喻为圣诞树，那么每个Fiber节点就是圣诞树上的挂件。

其中需要更新的节点就是亮的彩灯。

如何找到亮的彩灯（需要更新的节点）呢？

React的做法是：将需要更新的节点连接形成一条单链表。

查找时，只需要遍历这条单链表就行。就像圣诞树上的彩灯带一样。

#### C

React 团队想要发布新特性，底层也可以遍历整棵 Fiber 树

但是代码开发好了，又回滚，最后又被合进master，费劲周折。

可见，专业团队想达成 OKR 也不简单啊。

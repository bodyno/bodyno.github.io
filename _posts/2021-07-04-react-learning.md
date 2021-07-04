---
layout: post
title: "React源码学习笔记"
date: 2021-07-04
tags: [javascript]
categories: javascript
---

快速响应  
制约的瓶颈就是CPU和IO  
解决办法：异步可中断更新  
  
React15 使用的架构是 Reconciler + Renderer 两者交替运行 由于是同步的 所以视图不会出现中间状态  
React16 使用的架构是 Scheduler + Reconciler + Renderer 在Fiber上打上update 这个阶段是可中断的 最后在render阶段一次性提交  
实现了可以中断的异步更新  

代数效应 就是将副作用从函数中分离  
代数效应在hooks 中的体现 就是只需要写同步的逻辑 异步的数据获取和状态展示 可以完全交给React处理  
代数效应与Fiber架构有什么关系呢？  
Fiber利用了代数效应中可中断、恢复的思想来完成异步更新  
为什么不用generator  
可以中断，但不可以实现优先级  
  
为什么Fiber中的父节点引用叫return？  
因为React沿用V15版本中递归的逻辑 沿树一直向下，从父节点一直递到子节点，再从子节点【归】到父节点  
在Fiber中使用遍历的形式，也复用了这种思想，这就是为什么也叫return的原因  
  
首次调用React.render时会创建整个应用的根节点 FiberRootNode  
每一次调用会创建 RootFiber 当前应用的根节点  
FiberRootNode.current -> RootFiber  
然后进入首屏渲染的逻辑  
会从根节点创建一颗Fiber树 通过alternate 连接，方便共用一些属性  
这样就存在两颗Fiber树 一个叫current Fiber树 另一个叫workInProgress Fiber树  
  
Fiber的三层含义  
静态的数据结构  
动态的工作单元  
架构  
  
双缓存原理  
  
ReactElement 就是React.Element返回的结果  
ReactComponent 使用ClassComponent与FunctionComponent构建组件。  
  
beginWork completeWork  
执行beginWork 会创建并返回它的子fiber节点  
  
commit 对节点的修改就叫作 mutation  
分为before mutation mutation layout  
flushSyncCallbackQueue 要在开启concurrentMode 下才生效  
  
passive 的effect 就是function组件中useEffect对应的hooks    
useEffect会在commit阶段结束后以异步的方式执行  
update Effect 执行的是commitWork方法  
layout 阶段同步执行的是useLayoutEffect ComponentDidMount、ComponentDidUpdate  
  
打工人不能跳出宿命圈子，只能在圈子里打转并在他们定好的晋升规则下进步  
游戏王是怎么玩的  
  
每个节点都小于他后面的节点 就是小顶堆  
堆顶的任务就是过期时间最短的这个任务  
React 时间切片 怎么实现的 怎么模拟channel.postMessage
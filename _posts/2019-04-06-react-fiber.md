---
layout: post
title: "react16-fiber协调算法"
description: "react16 fiber协调算法的实现，fiberNode diff"
category: tech
tags: ['react']
---
{% include JB/setup %}

是的，都9012年了，我还在写fiber，因为我2个月前才认真看。。。

react是一个非常有革命性的框架，其开发者非常具有创新意识，在当年开创react时创造了view=f(data)，设计用函数式的模式让data直接反应视图。使用vdom diff来提高组件更新性能（虽不是首创）。在react15已经大范围普及的情况下，又做出了颠覆式的更新，react16 变更的东西非常多，很多地方就是推翻再来，譬如fiber 算法，不是局限于通过更优的算法来提升性能，而是结合浏览器现状，使用`requestAnimationFrame`和`requestIdleCallback`在浏览器空闲时碎片化分步更新内容，来达到更优的体验。

提到fiber算法的优化效果，想必都见过这张对比图：

![](http://s3.mogucdn.com/mlcdn/c45406/190406_5gkdlca7k824he218jca83109fb39_550x280.gif)
![](http://s3.mogucdn.com/mlcdn/c45406/190406_379jij3e66jkag26b94860hbe9d3l_550x280.gif)

# 背景

主要是为了适应动画场景，以及大规模组件更新的流畅性，改善体验。将原递归式同步协调算法，改为fiber算法的协调方式。

特点：
* 可拆分，可中断任务
* 可重用各分阶段任务，且可以设置优先级
* 可以在父子组件任务间前进后退切换任务
* render方法可以返回多元素（即可以返回数组）
* 支持异常边界处理异常

# fiberNode fiberTree

为了达到这些能力，react将底层更新单元的数据结构改成了链表结构。

以前的协调算法是递归调用，通过react dom 树级关系构成的栈递归。

而fiber是扁平化的链表的数据存储结构，通过child找子节点，return找父节点，sibling找兄弟节点。遍历从递归改为循环

通过fiberNode描述组件节点，return\child\sibling，将底层原栈结构改成链表结构，更加节省内存。而且链表结构在处理时刻随时暂停恢复，无需一气呵成阻塞ui。

![](https://s10.mogucdn.com/mlcdn/c45406/190406_7je839e1e4k7eg80h0bia1fabi67l_1306x1136.png)

Fiber算法的拆分单位是fiber（fiber tree上的一个节点），实际上就是按虚拟DOM节点拆，因为fiber tree是根据vDOM tree构造出来的，树结构一模一样，只是节点携带的信息有差异。实际上是vDOM node粒度的拆分（以fiber为工作单元），每个组件实例和每个DOM节点抽象表示的实例都是一个工作单元。工作循环中，每次处理一个fiber，处理完可以中断/挂起整个工作循环

# 更新过程

## reconciler和commit

React组件渲染分为两个阶段：reconciler、commit

- 协调reconcile：对Virtual DOM操作阶段，对应到新的调度算法中，就是通过 Diff Fiber Tree 找出要做的更新工作。这是一个js计算过程，计算结果可以被缓存，计算过程可以被打断，也可以恢复执行。
  所以，react介绍 Fiber Reconciler 调度算法时，有提到新算法具有可拆分、可中断任务的新特性，就是因为这部分的工作是一个纯js计算过程，所以是可以被缓存、被打断和恢复的

- 提交更新commit: 渲染阶段，拿到更新工作，提交更新并调用对应渲染模块（react-dom）进行渲染。为了防止页面抖动，该过程是同步且不能被打断。

在reconcile阶段的每个工作循环中，每次处理一个fiber，处理完可以中断/挂起整个工作循环。通过每个节点更新结束时向上归并effect list来收集任务结果，reconciliation结束后，根节点的effect list里记录了包括DOM change在内的所有side effect

在commit 阶段，`commitRoot`中遍历`nextEffect`，`invokeGuardedCallback(null, commitAllHostEffects, null)`;，根据`effect`的`effectTag`，进行对应的插入、更新、删除操作，根据tag不同，调用不同的更新方法，如class直接return,HostComponent和HostText才有对应的dom更新操作，通过`FiberNode.stateNode`找到对应的dom节点。例如commitUpdate中又会更新FiberProps和domProps。

## 异步调用的实现

异步任务调度有两种方式，主要是通过该任务的优先级进行判断，主要有两种：
1. animation（动画）：则会调用 `requestAnimationFrame` API 告诉浏览器，在下一次重绘之前调用该任务来更新动画
2. 其他异步任务：则会调用 `requestIdleCallback` API 告诉浏览器，在浏览器空闲时期依次调用任务，这就可以让开发者在主事件循环中执行后台或低优先级的任务，而且不会对像动画和用户交互等关键的事件产生影响

### 双缓冲技术

更新方式，更新过程中采用了双缓冲技术，通过存储的2颗fiber树处理：current、workInProgress，更新新树，丢弃老树

![](https://s10.mogucdn.com/mlcdn/c45406/190406_0g1cjlhfkk02ke3g8340lfbg26dij_1356x1012.png)

两颗 Fiber Tree：current、workInProgress，它们之间是通过每个Fiber节点上的alternate属性联系在一起。所有更新都是在workInProgress上进行操作，等更新完毕之后，再把current指针指向workInProgress，从而丢弃旧的Fiber Tree

## 过程概述

setState - scheduleWork - requestWork获得可用的协调计算时间 - performWork进入workLoop循环 -beginWork中进行fiberNode的diff获得变更的node节点记录到effectList，此过程中如果有shouldComponentUpdate方法调用后false则不进行下一步获得新node的比对 - commit阶段将effectList中所有需要更新的node取出，操作dom更新


具体更新过程：更新节点的props\state\context等属性，打effectTag - shouldComponentUpdate - 调用render()获得最新ReactElement 并为其绑定fiber节点，会尽力重用原节点 - 如果是新建节点直接mount，如果是更新类型则调用fiber协调算法 - 根据类型调用不同类型的diff算法 - 获得变更，打上effectTag，记录effectList  - 有child继续 - 没有child，将effectList合并到return，开始sibling作为下一个更新单元 - 如果没有时间了，等待下一次主线程空闲 - 如果没有sibling了，进入commit

commit过程：遍历root的effectList, 通过nextEffect对应的fiber节点的effectTag获得增删改的操作类型 ，node类型是class直接继续, 直到HostComponent和HostText才有对应的dom更新操作

## 过程详解

![](https://s10.mogucdn.com/mlcdn/c45406/190406_24idh37ad28ig61f94ea6ckb108g4_1302x472.png)
![](https://s10.mogucdn.com/mlcdn/c45406/190406_11i0i4bidlbihc486l7004ckagl5f_1294x130.png)

1. 第一部分从 `ReactDOM.render()` 方法开始，把接收的React Element转换为Fiber节点，并为其设置优先级，记录update等。这部分主要是一些数据方面的准备工作。
2. 第二部分主要是三个函数：`scheduleWork`、`requestWork`、`performWork`，即安排工作、申请工作、正式工作三部曲。React 16 新增的异步调用的功能则在这部分实现。
3. 第三部分是一个大循环，遍历所有的Fiber节点，通过Diff算法计算所有更新工作，产出 EffectList 给到commit阶段使用。这部分的核心是 beginWork 函数。

从`beginWork`来看，在`beginWork`阶段，`updateHostComponent`的时候会执行`reconcileChildFibers`或者`mountChildFibers(`初始化的时候)。主要分为两部分，一部分是对`Context`的处理，一部分是根据`fiber`对象的`tag`类型，调用对应的update方法（**vdom diff的步骤就是在这里**）：

1. 先更新classComponent的Instance，期间调用生命周期函数 
2. 根据shouldUpdate来判断是否需要更新 
3. 需要更新的话调用render()获得最新ReactElement 
4. 如果是新建节点直接mount，如果是更新类型则调用fiber协调算法
5. `reconcileChildFibers`函数中主要是根据newChild类型，调用不同的Diff算法：`reconcileSingleElement、reconcileSinglePortal、reconcileSingleTextNode、reconcileChildrenArray`，在子节点上添加 `effectTag`记录变更类型如`placement`
6. `reconcileSingleElement`的diff方式可参考：[https://react.jokcy.me/book/flow/reconcile-children/single.html](https://react.jokcy.me/book/flow/reconcile-children/single.html) 。array子元素群的diff方式：[https://react.jokcy.me/book/flow/reconcile-children/array.html](https://react.jokcy.me/book/flow/reconcile-children/array.html)
7. commit阶段做的事情是拿到`reconciliation`阶段产出的`EffectList`，即所有更新工作，提交这些更新工作并调用渲染模块（react-dom）渲染UI。

最后通过事件触发进入到commitAllHostEffects里进行dom操作步骤。

可以看下某次debug的函数调用栈：

![](https://s10.mogucdn.com/mlcdn/c45406/190406_6b06ll862cf5keh0c10l10kd7eka1_588x1096.png)

react16 更新元素的调用栈清爽的多，不想react15 因为递归能出现上百个调用栈，让你debug到迷失。

整个过程已经没有事务系统了，那之前用事务系统解决的问题如元素选中状态、生命周期的调用怎么处理的呢？在commitRoot里还是会有`prepareForCommit()`来处理如focus元素选中问题，也会有对应生命周期如`getSnapshotBeforeUpdate()`的调用

## 过程图

最后上一张网传的非常详细的流程导图：

![](https://s10.mogucdn.com/mlcdn/c45406/190406_01f3d1791ahh1ki92cbfgh1h75k06_1376x4096.png)


## 参考

- [React Fiber架构](https://zhuanlan.zhihu.com/p/37095662)
- [React16源码之React Fiber架构](https://juejin.im/post/5b7016606fb9a0099406f8de)
- [完全理解React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)
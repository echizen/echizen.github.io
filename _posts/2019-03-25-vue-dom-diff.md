---
layout: post
title: "vue源码-dom diff"
description: "vdom diff算法，vue dom diff"
category: tech
tags: ['vue']
---
{% include JB/setup %}

vue的组件更新过程经历dom diff和patch过程。本文就介绍vdom的diff算法。硬核预警。

# 核心代码

`src/core/vdom/patch.js`:

```javascript
 function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    let oldStartIdx = 0
    let newStartIdx = 0
    let oldEndIdx = oldCh.length - 1
    let oldStartVnode = oldCh[0]
    let oldEndVnode = oldCh[oldEndIdx]
    let newEndIdx = newCh.length - 1
    let newStartVnode = newCh[0]
    let newEndVnode = newCh[newEndIdx]
    let oldKeyToIdx, idxInOld, elmToMove, refElm

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    const canMove = !removeOnly

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx] // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx]
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)
        oldStartVnode = oldCh[++oldStartIdx]
        newStartVnode = newCh[++newStartIdx]
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue)
        oldEndVnode = oldCh[--oldEndIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue)
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))
        oldStartVnode = oldCh[++oldStartIdx]
        newEndVnode = newCh[--newEndIdx]
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue)
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
        oldEndVnode = oldCh[--oldEndIdx]
        newStartVnode = newCh[++newStartIdx]
      } else {
        if (isUndef(oldKeyToIdx)) oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
        idxInOld = isDef(newStartVnode.key) ? oldKeyToIdx[newStartVnode.key] : null
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
          newStartVnode = newCh[++newStartIdx]
        } else {
          elmToMove = oldCh[idxInOld]
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !elmToMove) {
            warn(
              'It seems there are duplicate keys that is causing an update error. ' +
              'Make sure each v-for item has a unique key.'
            )
          }
          if (sameVnode(elmToMove, newStartVnode)) {
            patchVnode(elmToMove, newStartVnode, insertedVnodeQueue)
            oldCh[idxInOld] = undefined
            canMove && nodeOps.insertBefore(parentElm, elmToMove.elm, oldStartVnode.elm)
            newStartVnode = newCh[++newStartIdx]
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)
            newStartVnode = newCh[++newStartIdx]
          }
        }
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
    }
  }

```

## 案例

光看代码会懵逼，不妨结合几个典型例子跑一下：

循环中，每一轮结束记录的是当前状态的dom顺序

### 例1：倒序

old：a b c d e

now：e d c b a

```
使用规则：
sameVnode(oldStartVnode, newEndVnode) ->
nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm))

#1
old: abcde
new: edcba
oS:a  oE:e  nS:a nE:b
res：bcdea -  oS  `a` moved after oE `e`

#2
old: a bcde
new: edcb a
oS:b  oE:e  nS:e nE:b
res：cdeba -   oS `b` moved after oE `e`

#3
old: ab cde
new: edc ba
oS:c  oE:e  nS:e nE:c
res：decba -   oS `c` moved after oE `e`

#4
old: abc de
new: ed cba
oS:d  oE:e  nS:e nE:d
res：edcba -   oS `d` moved after oE `e`

#5
old: abcd e
new: e dcba
oS:e  oE:e  nS:e nE:e
rule: `sameVnode(oldStartVnode, newStartVnode), patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res：edcba

```

### 例2：换序

old: a b c d e
new: a c e b d

```
#1
old: abcde
new: acebd
oS:a  oE:e  nS:a  nE:d
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res：abcde 

#2
old: a bcde
new: a cebd
oS:b  oE:e  nS:c  nE:d
rule: 进入`createKeyToOldIdx`的key值比较环节。对c和old进行key值比较，发现与第二个相等，则把o2移到o1前面。oldCh[2] = undefined, 此时，oldStartIdx === 1，后面的循环中，oldCh[1]会导致 isUndef(oldStartVnode)，直接oldStartIdx++
res：acbde 
oldCh[2] = undefined

#3
(U表示undefined)
old: a bUde
new: ac ebd
oS:b  oE:e  nS:e nE:d
rule: sameVnode(oldEndVnode, newStartVnode) ->  nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
res: acebd -> 上一次结果acbde 中：oE `e` moved before oS `b`

#4
(U表示undefined)
old: a bUd e
new: ace bd
oS:b  oE:d  nS:b nE:d
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res: acebd 

#5
(U表示undefined)
old: ab Ud e
new: aceb d
oS:U  oE:d  nS:d nE:d
rule: `isUndef(oldStartVnode)`
res: acebd 

#6
(U表示undefined)
old: abU d e
new: aceb d
oS:d  oE:d  nS:d nE:d
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res: acebd 

```

### 例3：增加元素

old: a b c
new: a d b c

```
#1
old: abc
new: adbc
oS:a  oE:c  nS:a  nE:c
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res：abc

#2
old: a bc
new: a dbc
oS:b  oE:c  nS:d  nE:c
rule: `isUndef(idxInOld)) -> createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm)`d与任何old都不相等，则createElm, 将nS:d 嵌入到oS:b前面
res：adbc


#3
old: a bc
new: ad bc
oS:b  oE:c  nS:b  nE:c
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res：adbc

#4
old: ab c
new: adb c
oS:c  oE:c  nS:c  nE:c
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res：adbc

```

### 例4：增加元素

old: a b c
new: a c

```
#1
old: abc
new: ac
oS:a  oE:c  nS:a  nE:c
rule: `sameVnode(oldStartVnode, newStartVnode) -> patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue)`
res：abc

#2
old: a bc
new: a c
oS:b  oE:c  nS:c  nE:c
rule: `sameVnode(oldEndVnode, newStartVnode) -> nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)`
res：acb -> oE `c` moved before oS `b`


#3
old: a b c
new: ac
oS:b  oE:b  newStartIdx = 2     newEndIdx = 1 , newStartIdx > newEndIdx, 结束循环
移除oldStartIdx:1 至 oldEndIdx:1 之间的元素即b
removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx)
res: 13

```

特别说明下`createKeyToOldIdx`并进行后续查找的作用：双端比较完后，双端没有相同的节点，不代表就没有和中间元素相同的节点，所以会拿newStart和old队列里比较找出key相同的做下一步比较

这个算法过程是不是清晰多了，那么为啥要用这种算法呢，为啥一直在对比oldStartVnode、oldEndVnode、newStartVnode、newEndVnode这些值呢？

# vdom diff算法

先聊聊vdom，vdom是为了减轻性能压力。dom是昂贵的，昂贵的一方面在dom本身的重量，dom节点在js里的描述是个非常复杂属性很多原型很长的超级对象，另一方面是浏览器渲染进程和js进程是独立分离的，操作dom时的通信和浏览器本身需要重绘的时耗都是很高的。

所以大家机智的搞了个轻量的vdom去模拟dom，vdom每个节点都只挂载js操作的必要属性，每次组件update时都先操作vdom，通过vdom的比对，得到一个真实dom的需要操作的集合。整个机制是在JavaScript层面计算，在完成之前并不会操作DOM，等数据稳定之后再实现精准的修改。

vdom是vue2才引入的，在依赖收集的基础上加持vdom diff，直接定位所需diff的组件，类似react dom diff + shouldComponentUpdate优化处理，快速跳过无需diff的组件。性能得到大幅提升。

另外，vdom是抽象dom，可跨端，web和native可以根据抽象层实现自己的render层。这就达到了让核心dom diff算法层统一逻辑，但是渲染层交给不同端自己实现。

## vdom diff算法优化

传统diff算法通过循环递归对节点进行依次对比效率低下，算法复杂度达到O(N^3)，主要原因在于其追求完全比对和最小修改，而React、Vue则是放弃了完全比对及最小修改，才实现从O(N^3) => O(N)。

优化措施有：

- 分层diff：不考虑跨层级移动节点，让新旧两个VDOM树的比对无需循环递归（复杂度大幅优化，直接下降一个数量级的首要条件）。这个前提也是Web UI 中 DOM 节点跨层级的移动操作特别少，可以忽略不计。
- 找到diff的关键属性，发生以下情况则跳过比对，变为插入或删除操作：
    - 组件的Type(Tagname)不一致，原因是绝大多数情况拥有相同type的两个组件将会生成相似的树形结构，拥有不同type的两个组件将会生成不同的树形结构，所以type不一致可以放弃继续比对。
    - 列表组件的Key不一致，旧树中无新Key或反之。毕竟key是元素的身份id，能直接对应上是否是同一个节点。
- 对触发了getter/setter 的组件进行diff，精准减少diff范围

[Virtual DOM 背后的秘密（Diff 篇）](https://zhuanlan.zhihu.com/p/36500459)

## 双端比较算法内容

dom diff算法其实有很多很多种，vue采用了`snabbdom`的双端比对算法，算法复杂度仍为O(N)。

可了解下snabbdom本身的代码：[https://github.com/snabbdom/snabbdom/blob/b4d69aea97a7e57e8642d5401f9e36f6e566294f/src/snabbdom.ts#L179](https://github.com/snabbdom/snabbdom/blob/b4d69aea97a7e57e8642d5401f9e36f6e566294f/src/snabbdom.ts#L179)。vue2是完全使用了这部分内容。

可以看到，一款优秀的框架也不是啥都原创，而是在一个好的设计上不断吸收并进，不断使用更优秀的东西优化发展自己。

## vue和react 15的vdom diff算法优劣

vue和react15的vdom diff算法是不一样的。

### react 15 dom diff

简要描述下react的dom diff过程：

![](https://pic4.zhimg.com/80/c0aa97d996de5e7f1069e97ca3accfeb_hd.png)

![](https://pic1.zhimg.com/80/7b9beae0cf0a5bc8c2e82d00c43d1c90_hd.png)

if (child._mountIndex < lastIndex)，则进行节点移动操作

看当前遍历到的新节点在老树中的位置_mountIndex与老树中最大访问过的下标lastIndex比较。（lastIndex会记录老树中的访问过的元素的最大位置或者是新插入的元素的位置）
只关注老树中当前访问过的位置lastIndex的移动状况，如果_mountIndex大于这个值，则表示在访问点后面，暂时不操作。只有小于了，才去操作移动。这种思想保证永远只有一个方向向后移动，提高效率。但是如果是最后一个节点移动到第一个，则效率会很低。

更多react 15 diff解释可查看[React 源码剖析系列 － 不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379)

### 对比


直接用“章辰-转转”知乎文章里整理的这张实际案例的对比图：

![](https://pic4.zhimg.com/80/v2-3e18884058de8cbea2a9b6174b73cce3_hd.jpg)

图例：
- 绿色表示更新的节点，黑色表示复用的节点
- 红色表示新插入的节点，逗号之间表示老节点被删除或被移动
- 黄/蓝色表示移动的节点，对应理论最优不动/移动解
- 前两行在生命周期的二次渲染中上色，后三行则在首次渲染后手工上色
- `,`作为分隔符，以便看起来更像数组原本的结构

**可以看出react 15这种算法非常不适合倒序输出，全部向后移。而vue的双端比较法比较折中适合大多数场景，表现是向前移动。**

所以针对这种特征

* 建议，在开发组件时，保持稳定的 DOM 结构会有助于性能的提升；
* 建议，在开发过程中，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，在一定程度上会影响 React 的渲染性能。

# vdom diff是否一定性能更好

这个问题在知乎上有非常好的讨论，直接摘录有意义的片段：

> 1. 原生 DOM 操作 vs. 通过框架封装操作。这是一个性能 vs. 可维护性的取舍。框架的意义在于为你掩盖底层的 DOM 操作，让你用更声明式的方式来描述你的目的，从而让你的代码更容易维护。**没有任何框架可以比纯手动的优化 DOM 操作更快，因为框架的 DOM 操作层需要应对任何上层 API 可能产生的操作，它的实现必须是普适的**。针对任何一个 benchmark，我都可以写出比任何框架更快的手动优化，但是那有什么意义呢？在构建一个实际应用的时候，你难道为每一个地方都去做手动优化吗？出于可维护性的考虑，这显然不可能。框架给你的保证是，你在不需要手动优化的情况下，我依然可以给你提供过得去的性能。不要天真地以为 Virtual DOM 就是快，diff 不是免费的，batching 么 MVVM 也能做，而且最终 patch 的时候还不是要用原生 API

2. 对 React 的 Virtual DOM 的误解。如果没有 Virtual DOM，每次有变动就整个重新渲染整个应用简单来想就是直接重置 innerHTML。在一个大型列表所有数据都变了的情况下，重置 innerHTML 其实是一个还算合理的操作... 真正的问题是在 “全部重新渲染” 的思维模式下，即使只有一行数据变了，它也需要重置整个 innerHTML，这时候显然就有大量的浪费。
  -innerHTML:  render html string O(template size) + 重新创建所有 DOM 元素 O(DOM size)-Virtual DOM: render Virtual DOM + diff O(template size) + 必要的 DOM 更新 O(DOM change)
  
  Virtual DOM render + diff 显然比渲染 html 字符串要慢，但是！它依然是纯 js 层面的计算，比起后面的 DOM 操作来说，依然便宜了太多。可以看到，innerHTML 的总计算量不管是 js 计算还是 DOM 操作都是和整个界面的大小相关，但 Virtual DOM 的计算量里面，只有 js 计算和界面大小相关，DOM 操作是和数据的变动量相关的。前面说了，和 DOM 操作比起来，js 计算是极其便宜的。这才是为什么要有 Virtual DOM：它保证了 1）不管你的数据变化多少，每次重绘的性能都可以接受；2) 你依然可以用类似 innerHTML 的思路去写你的应用。

by 尤雨溪


在尤雨溪看来：Virtual DOM 真正的价值从来都不是性能，而是它 1) 为函数式的 UI 编程方式打开了大门；2) 可以渲染到 DOM 以外的 backend，比如 ReactNative。

先说innerHTML还是Document.createElement + appendChild谁更优劣（举例增加dom的api，实际上不同情况自然会调用不同的dom操作api），毕竟场景不同效果不一样，大面积的改动前者性能更优，而小范围不连贯的dom节点更新后面的方式精确改动dom显然性能更好。

再者，同样都是Document.createElement + appendChild去精确改动的情况下，react还多了一步dom diff的过程，虽然js执行快，但怎么也是多了一步，自然比不上直接去patch阶段的操作dom。

再退一步，同样都是dom diff+ patch操作dom的方案，你通过手动优化在自己的场景下总能写出性能更高的dom操作方式。但是你不能为每个场景都去手写优化。如果在vdom diff上遇到无法优化的性能问题了，比较合理的做法应该是针对这一类特定场景替换成更优的dom diff + patch方案。

更多讨论查看：[网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么?](https://www.zhihu.com/question/31809713)

# 参考

在研究vdom diff算法时参考了很多有价值的文章，推进理解上了一个深度，统一列举下：

- [React 源码剖析系列 － 不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379)
- [Virtual DOM 背后的秘密（DOM 篇）](https://zhuanlan.zhihu.com/p/36259218)
- [Virtual DOM 背后的秘密（Diff 篇）](https://zhuanlan.zhihu.com/p/36500459)
- [网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么?](https://www.zhihu.com/question/31809713), 尤雨溪、胡子大哈、司徒正美、罗志宇的回答都非常有价值，知乎当年的氛围真的是很好的，可现在，唉，挽尊...
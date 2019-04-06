---
layout: post
title: "react16-fiberNode diff"
description: "react16-fiberNode diff, fiber diff算法, react16 vdom diff"
category: tech
tags: ['react']
---
{% include JB/setup %}

在了解了链表增删改查、fiberNode结构、react 16组件更新流程后，我们可以来具体看看fiberNode diff算法的具体实现了。


# reconcile流程

先来回顾下reconcile的流程：

1. 如果当前节点不需要更新，直接把子节点clone过来，跳到5；要更新的话打个tag
2. 更新当前节点状态（props, state, context等）
3. 调用shouldComponentUpdate()，false的话，跳到5
4. 调用render()获得新的子节点，并为子节点创建fiber（创建过程会尽量复用现有fiber，子节点增删也发生在这里）
5. 如果没有产生child fiber，该工作单元结束，把effect list归并到return，并把当前节点的sibling作为下一个工作单元；否则把child作为下一个工作单元
6. 如果没有剩余可用时间了，等到下一次主线程空闲时才开始下一个工作单元；否则，立即开始做
7. 如果没有下一个工作单元了（回到了workInProgress tree的根节点），进入pendingCommit状态

react在比对子fiber时，会根据子fiber类型在`reconcileChildFibers`里有不同的比对方案：`reconcileSingleElement`、`reconcileSinglePortal`、`reconcileSingleTextNode`、`reconcileChildrenArray`、`reconcileChildrenIterator`

# reconcileChildrenArray

具体看看子元素是数组时，`reconcileChildrenArray`这个最为复杂的比对场景

## 源码

packages/react-reconciler/src/ReactChildFiber.js:

```javascript
 function reconcileChildrenArray(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChildren: Array<*>,
    expirationTime: ExpirationTime,
  ): Fiber | null {

    if (__DEV__) {
      // First, validate keys.
      let knownKeys = null;
      for (let i = 0; i < newChildren.length; i++) {
        const child = newChildren[i];
        knownKeys = warnOnInvalidKey(child, knownKeys);
      }
    }

    let resultingFirstChild: Fiber | null = null;
    let previousNewFiber: Fiber | null = null;

    let oldFiber = currentFirstChild;
    let lastPlacedIndex = 0;
    let newIdx = 0;
    let nextOldFiber = null;
    for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
      if (oldFiber.index > newIdx) {
        nextOldFiber = oldFiber;
        oldFiber = null;
      } else {
        nextOldFiber = oldFiber.sibling;
      }
      const newFiber = updateSlot(
        returnFiber,
        oldFiber,
        newChildren[newIdx],
        expirationTime,
      );
      if (newFiber === null) {
        if (oldFiber === null) {
          oldFiber = nextOldFiber;
        }
        break;
      }
      if (shouldTrackSideEffects) {
        if (oldFiber && newFiber.alternate === null) {
          // We matched the slot, but we didn't reuse the existing fiber, so we
          // need to delete the existing child.
          deleteChild(returnFiber, oldFiber);
        }
      }
      // newFiber.index = newIndex;。lastPlacedIndex的作用没看到
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
      oldFiber = nextOldFiber;
    }

    if (newIdx === newChildren.length) {
      // We've reached the end of the new children. We can delete the rest.
      deleteRemainingChildren(returnFiber, oldFiber);
      return resultingFirstChild;
    }

    if (oldFiber === null) {
      // If we don't have any more existing children we can choose a fast path
      // since the rest will all be insertions.
      for (; newIdx < newChildren.length; newIdx++) {
        const newFiber = createChild(
          returnFiber,
          newChildren[newIdx],
          expirationTime,
        );
        if (!newFiber) {
          continue;
        }
        lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
      return resultingFirstChild;
    }

    // Add all children to a key map for quick lookups.
    const existingChildren = mapRemainingChildren(returnFiber, oldFiber);

    // Keep scanning and use the map to restore deleted items as moves.
    for (; newIdx < newChildren.length; newIdx++) {
      // 从existingChildren找出newIdx相同的元素复用更新
      const newFiber = updateFromMap(
        existingChildren,
        returnFiber,
        newIdx,
        newChildren[newIdx],
        expirationTime,
      );
      if (newFiber) {
        if (shouldTrackSideEffects) {
          if (newFiber.alternate !== null) {
            // The new fiber is a work in progress, but if there exists a
            // current, that means that we reused the fiber. We need to delete
            // it from the child list so that we don't add it to the deletion
            // list.
            existingChildren.delete(
              newFiber.key === null ? newIdx : newFiber.key,
            );
          }
        }
        lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
        if (previousNewFiber === null) {
          resultingFirstChild = newFiber;
        } else {
          previousNewFiber.sibling = newFiber;
        }
        previousNewFiber = newFiber;
      }
    }

    if (shouldTrackSideEffects) {
      // Any existing children that weren't consumed above were deleted. We need
      // to add them to the deletion list.
      existingChildren.forEach(child => deleteChild(returnFiber, child));
    }

    return resultingFirstChild;
  }
```

## 解析

讲真，第一二遍看我都是懵的，特别不理解为啥要做两次循环及第一次循环的作用。

还是老方法，慢点创造个实例去debug，快点自己模拟一下跟着代码走一遍流程。具体case就不放了。解释如下：

第一次遍历：先遍历新元素链，找到index相同的元素，主要判断key是否相同，相同的话update老节点生成新节点newFiber。如果遍历到的相同index而元素不相等则结束第一个循环。

第一次遍历完后：
- 新链结束老链没有：把老链中剩余的fiber都删除
- 老链结束新链没有：把新链中剩下的都插入
- 其他情况：把老链按key放入map里，遍历新链，从老链中找和新链key相同的fiber，更新成newfiber，供后面赋值给previousNewFiber构成新链中的一环并进行下一轮循环，也是个链表元素移动的过程，再将老链中该元素删除。遍历处理完所有newChildren生成新链后，删除老链中剩下的元素【核心步骤】

看下来还会有这些问题：

### 返回为啥是一个fiber节点？

这里比较疑惑的是，因为原本要diff的是一个列表，常规思路diff完后应该返回更新的fiber元素列表，为啥就返回了一个元素？reconcileChildFibers最终的返回是当前节点的第一个孩子节点。reconcileChildrenArray返回的也是一个节点，由于是链表结构，知道第一个节点，就能知道整个链路。

### effect的作用

在fiber diff出结果后的操作代码里，譬如`createChild`、`placeChild`、`deleteChild`这些方法都没有直接操作fiberNode的增删或者dom的增删，而是去操作了effect。譬如`deleteChild`的代码：

```javascript
function deleteChild(returnFiber: Fiber, childToDelete: Fiber): void {
  if (!shouldTrackSideEffects) {
    // Noop.
    return;
  }
  // Deletions are added in reversed order so we add it to the front.
  // At this point, the return fiber's effect list is empty except for
  // deletions, so we can just append the deletion to the list. The remaining
  // effects aren't added until the complete phase. Once we implement
  // resuming, this may not be true.
  const last = returnFiber.lastEffect;
  if (last !== null) {
    last.nextEffect = childToDelete;
    returnFiber.lastEffect = childToDelete;
  } else {
    returnFiber.firstEffect = returnFiber.lastEffect = childToDelete;
  }
  childToDelete.nextEffect = null;
  childToDelete.effectTag = Deletion;
  }
```

那么这样操作effect和打effectTag的作用是？

effect是side effect，代表将要做的DOM change及；诶性。每个workInProgress tree节点上都有一个 effect list 用来存放diff结果，当前节点更新完毕会向上merge effect list（queue收集diff结果），供后续commit阶段，对整个fiberTree 映射的dom 在effect list里的做增、删、更新处理。

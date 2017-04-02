---
layout: post
title: "react合成事件和原生事件的阻止冒泡"
description: "react合成事件和原生事件的阻止冒泡"
category: tech
tags: [react, js]
---
{% include JB/setup %}

以下材料多来源于小伙伴们讨论时给出。

我们知道，react拥有自己的事件系统，所使用的`SyntheticEvent`（合成事件）是和原生事件有区别的。


> Your event handlers will be passed instances of SyntheticEvent, a cross-browser wrapper around the browser's native event. It has the same interface as the browser's native event, including stopPropagation() and preventDefault(), except the events work identically across all browsers.
If you find that you need the underlying browser event for some reason, simply use the nativeEvent attribute to get it.

[https://facebook.github.io/react/docs/events.html](https://facebook.github.io/react/docs/events.html)

## 由阻止事件冒泡引发的问题

合成事件不能阻止原生事件: [https://jsbin.com/veqofetoni/edit?html,js,output](https://jsbin.com/veqofetoni/edit?html,js,output)

原生事件可以阻止合成事件: [https://jsbin.com/paqawokeho/edit?html,js,console,output](https://jsbin.com/paqawokeho/edit?html,js,console,output)

## 合成事件不能阻止原生事件，原生事件可以阻止合成事件

react 和jQuery 一样，都是封装了自己的事件系统。 react 默认事件代理的方式，实际上没有任何冒泡的过程，需要程序自己模拟冒泡的操作。

在jQuery 和React 中事件代理都会自己维护一个从下到上的  Queue的监听队列，一个元素事件触发时，会从下往上执行这个队列中的监听函数，模拟冒泡的过程。

另外React是维护自己的事件队列， React 的 stopPropagation 只会处理自己的队列冒泡，因为代理到最外层节点上如document，真正点击的节点并触发任何事件，`event.nativeEvent.stopPropagation()` 实际上是在最外层节点上调用了原生的 stopPropagation， 只组织了body的冒泡。

**所以在react事件回调函数中调用stopPropagation，react自己维护的事件队列里阻止了余下队列里回调函数的执行，进而成功阻止了react合成事件的冒泡，但是在调用`event.nativeEvent.stopPropagation`时，由于nativeEvent是原生事件对象，react是将事件代理到在最外层元素上的，所以调用这个只能阻止最外层元素如document上的事件冒泡，且由于只能阻止当前事件的当前监听器冒泡执行，也就是react合成事件的冒泡，几乎没有实际意义。调用`event.nativeEvent.stopImmediatePropagation`，由于其能组织该元素绑定的后序相同类型事件的监听函数的执行，所以对于代理到document上的原生事件也会被组织执行，达到了组织合成事件和原生事件冒泡的作用，但是这个要求是大家都是事件代理到最外层元素如document上**

但是如果用原生的事件的 stopPropagation， 会阻止事件向上冒泡到最外层节点，这样React的事件代理就无法工作了，最外层节点接受不到冒泡上来的事件，所以原生的stopPropagation 可以阻止React的 事件，但是原生事件也必须绑在作用元素上，而不是通过事件代理机制。

## react事件回调函数中event上的阻止冒泡作用

- `event.stopPropagation`：在react事件回调函数中调用能阻止react合成事件的冒泡
- `event.nativeEvent.stopPropagation`： 基本没啥实际作用，阻止的是代理到根元素（如document）的事件。
- `event.nativeEvent.stopImmediatePropagation`： 阻止调用相同事件的其他侦听器，除了该事件的冒泡行为被阻止之外(event.stopPropagation方法的作用),该元素绑定的后序相同类型事件的监听函数的执行也将被阻止.所以可以阻止和react一样代理在document上的事件，如jquery中的
`$(document).on('click', function(){});`

## 源码解读：

[https://github.com/facebook/react/blob/b1b4a2fb252f26fe10d29ba60d85ff89a85ff3ec/src/renderers/shared/shared/event/SyntheticEvent.js#L136-L153](https://github.com/facebook/react/blob/b1b4a2fb252f26fe10d29ba60d85ff89a85ff3ec/src/renderers/shared/shared/event/SyntheticEvent.js#L136-L153)

[https://github.com/facebook/react/blob/b1b4a2fb252f26fe10d29ba60d85ff89a85ff3ec/src/renderers/shared/shared/event/EventPluginUtils.js#L101-L125](https://github.com/facebook/react/blob/b1b4a2fb252f26fe10d29ba60d85ff89a85ff3ec/src/renderers/shared/shared/event/EventPluginUtils.js#L101-L125)

[https://github.com/facebook/react/blob/a749f4fb63657a3af1502047a5af876ebabb0e64/src/renderers/shared/shared/event/EventPropagators.js#L63-L88](https://github.com/facebook/react/blob/a749f4fb63657a3af1502047a5af876ebabb0e64/src/renderers/shared/shared/event/EventPropagators.js#L63-L88)

[https://github.com/facebook/react/blob/b1b4a2fb252f26fe10d29ba60d85ff89a85ff3ec/src/renderers/shared/shared/ReactTreeTraversal.js#L97-L110](https://github.com/facebook/react/blob/b1b4a2fb252f26fe10d29ba60d85ff89a85ff3ec/src/renderers/shared/shared/ReactTreeTraversal.js#L97-L110)

## 总结

所以在使用过程中，一定要清晰意识到原生事件和合成事件，分开管理。

react合成事件的stopPropagation只对react合成事件有效，可能干扰到代理在最外层元素上的其他事件系统的事件。

原生事件中非代理到最外层元素的事件模式调用stopPropagation会组织react合成事件的冒泡。另外，原生事件的监听需要在组件销毁时`componentDidUnmount`里去除监听，以免发生内存泄露。

参考：

[https://dlinau.wordpress.com/2015/09/16/avoid-mixing-reacts-event-system-with-native-dom-event-handling/](https://dlinau.wordpress.com/2015/09/16/avoid-mixing-reacts-event-system-with-native-dom-event-handling/)
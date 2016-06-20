---
layout: post
title: "使用requestAnimationFrame提高性能"
description: "使用requestAnimationFrame提高性能"
category: tech
date: 2016-03-26 23:12:43
tags: [性能]
---
{% include JB/setup %}

## requestAnimationFrame是干什么的

根据MDN的介绍，window.requestAnimationFrame()这个方法是用来在页面重绘之前，通知浏览器调用一个指定的函数，以满足开发者操作动画的需求。如果你想做逐帧动画的时候，你应该用这个方法。这就要求你的动画函数执行会先于浏览器重绘动作。通常来说，被调用的频率是每秒60次，但是一般会遵循W3C标准规定的频率。

## 为什么使用requestAnimationFrame

如果你有连续的动画效果需要实现，不使用requestAnimationFrame会有卡顿出现。requestAnimationFrame能在浏览器在页面重绘之前执行你的动画函数，这样你的动画每秒渲染60次，不会有视觉上的卡顿出现。

在`requestAnimationFrame`出现之前，大家都用`setTimeOut`和`setInterval`，但是这2个函数受到js引擎时间处理器的影响，如果时间到了，但是执行栈中还有队列排在动画函数之前，动画函数依旧不能执行，而每个间隔执行栈中的队列是不确定的也是程序员未知被浏览器管理的，所以这个很难保证动画连贯。

requestAnimationFrame不一样，使用它是将动画执行时间交给了浏览器，浏览器管理系统能够保证每帧都执行传递的函数参数。


## 使用

	requestAnimationFrame(function(){
	     // write you code
	})
	
## 举个🌰
之前写过一个功能就是通过touch事件模拟滚动，这时候当监听到touchMove到一定范围后，通过translateX来改变元素位置模拟滚动。因为touchMove持续被触发，所以改变translateX的函数持续被触发，要的是一个连贯的滚动动画效果。

	requestAnimationFrame(function(){
	     $(element).css(styles);
	})

不使用requestAnimationFrame：
![image](https://echizen.github.io/assets/blog-img/requestAnimationFrame1.png)

使用requestAnimationFrame：
![image](https://echizen.github.io/assets/blog-img/requestAnimationFrame2.png)

通过chrome timeLine的分析可以看到，红色长帧明显减少，性能提升明显。

关于chrome timeLine的小tips：
分析导致红色长帧的代码是，从调用栈最顶层看起，如果使用的是别人的库，一般都是顶层的调用语句有问题，左键右键切换移动每一个函数调用位置查看语句定位。

![image](https://echizen.github.io/assets/blog-img/requestAnimationFrame3.png)
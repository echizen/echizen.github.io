---
layout: post
title: "手机端禁止滚动"
description: "overflow:hidden在手机端失效，手机端如何禁止模态框背景滚动"
category: tech
tags: [css, pro]
---
{% include JB/setup %}

更新纠正：
bootstrap的模态框在手机端依旧能滚动，是因为它使用了position:fixed属性，改属性在手机上支持性很差，特别是iphone。bootsrap使用的是position:fixed和display:none的结合改变来达到不能滚动。framework7使用的是position:absolute和visibility: hidden来达到效果的，在手机端更有效。

我在做PC端模态框弹出时，禁止背景内容滚动的方式是在页面中加入一个div(最好作为body的直接子元素)，假设他的class为oveflow-bg，然后给`.oveflow-bg`以下样式：

	position: absolute;
  	left: 0;
  	top: 0;
  	width: 100%;
  	height: 100%;
  	background: rgba(0,0,0,.4);
  	z-index: 9999;
  	visibility: hidden;
  	
当模态框弹起时给body`overflow:hidden`的属性，将`.oveflow-bg`的改为`visibility: visible;`.

这种方式在页面结构简单时有效。

-----------------

或者和bootstrap一样，给模态框的最外层div以下属性：

	.modal {
	  position: fixed;
	  top: 0;
	  right: 0;
	  bottom: 0;
	  left: 0;
	  z-index: 1050;
	  display: none;
	  overflow: hidden;
	  -webkit-overflow-scrolling: touch;
	  outline: 0;
	  display: none;
	}
	
当模态框调出时，给body`overflow:hidden`的属性，给modal添加属性：

	display: block;
  	overflow-x: hidden;
  	overflow-y: auto;
	

这种方式在手机端一点作用也没有

-----------------------

今天用framework7时它的做法非常有效。如果我说他是利用颜色来做的（其实是透明度）结合visibility。

也是一个body的直接子元素：

	.popup-overlay, .preloader-indicator-overlay {
	  position: absolute;
	  left: 0;
	  top: 0;
	  width: 100%;
	  height: 100%;
	  background: rgba(0,0,0,.4);
	  z-index: 10600;
	  visibility: hidden;
	  opacity: 0;
	  z-index: 10200;
	}
	
模态框打开时添加以下属性：

	visibility: visible;
  	opacity: 1;
  	
我把开始见感到非常神奇啊，透明度可以影响这个，当我在chrome下审查元素时才发现，.popup-overlay的`opacity: 1;`覆盖在元素上时，获取不到其他元素了，也就是没有用`point-event:none`，也导致鼠标事件无效。

**结论：元素的透明度为0时，相当于不存在，鼠标事件对其失效。（鼠标事件和移动端触摸事件相同）**


这就很好解释原理了，这种方式是一开始就在屏幕上放上了一个满屏的popup-overlay，但是它的opacity=0,visibility: hidden;不会影响其他元素正常被用户点击，但是当他的visibility: visible;时，用户的触摸事件全都在该元素上生效，屏幕便无法滚动了。

另外：操作过程中，我把该元素.popup-overlay的颜色值调为：`clolor:transparent`,他下面所有的元素都可已被点击了，这其实和opacity=0同理。

小小颜色，大大妙用。
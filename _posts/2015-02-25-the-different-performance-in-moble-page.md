---
layout: post
title: "移动开发的几个坑"
description: "the different performance in moble page,移动设备页面注意事项，历史回退、position:fixed、overflow:hidden、scroll在移动端的不同于网页的表现"
category: tech
tags: [basic , js ,html, css, 前端]
---
{% include JB/setup %}

在写移动端界面时遇到了一些坑，很多是移动端浏览器本身就不同的表现，但是因为没经验，一直以为是自己代码的问题。记录自己遇到的几个坑，防止二度跳坑。

## positon:fixed

可以说这是一个老大难的问题，不仅ios系统有，某些安卓机也有。ios6及以下肯定有这个问题，ios7的一些版本修复了这个bug，但是到了ios8至ios8.1.3（写这篇时的版本）都还有这个问题。andriod的情况比ios好，andriod2.X以上版本基本都支持了。

这其实是浏览器的bug，但是浏览器不支持这个是有理由的。`fixed`属性实在太消耗浏览器体力：我们所看到的页面其实viewport区域是不变的，滑动是是让内容在viewport区域刷过，就相当于page是一张纸，viewport是一个带有一定面积透明可见的黑盒子，我们在盒子里抽纸，纸在移动，我们只能从盒子的透明部分看到纸上的内容，纸本身是一个整体，然而fixed的部分属于纸这个整体却要脱离这个整体粘在盒子的透明部分上。。。只要你在抽纸（滑动），浏览器只好不辞辛苦的实时计算，将fixed的部分在纸上移动，挪到合适的部位。我们经常看到fixed的部分在滑动时乱窜。这在性能比不上电脑的手机上是很消耗的一种行为，所以很多系统的浏览器都对fixed支持不好。

这个问题的解决方案一般是一些小技巧，譬如说由于ios下fixed错位发生在键盘弹起时，所以一般在input、textarea的focus事件发生时，让fixed的元素"display:none"掉或变成absolutely的定位，等到输入框发生blur事件时再变回原样，但是absolutely不是那么好用的，此时你可能为了可视区域不跳动，还得设元素的offsetTop的值。

如果你的fixed的元素中包含输入框，就像这样，那你就悲催了，小技巧不能用了。

![image](https://echizen.github.io/assets/blog-img/QQ20150225-1.png)

其实iscroll和 Sencha Touch等一些框架已经做到了fixed的真正的效果，是通过js实时计算模拟的，但是就为了fixed元素引入这么重的框架，显然是不合适的。惭愧的是我也没有认真研究透这是怎么工作的。

## overflow:hidden

		<html>
		    <head></head>
		    <body>
		        <section class="fullScreen"></section>
		    </body>
		</html>
	
这种结构的DOM，如果配上一下css，overflow:hidden是生效的。

	html,body,#fullScreen{
        width: 100%;
        height: 100%;
        overflow: hidden;
    }
    
  但是如果这个`overflow:hidden`是js动态增添的，则无效，手机端浏览器会对动态增添的`overflow:hidden`属性熟视无睹。即使注明框架bootstrap的modal后的遮罩层也没有解决这个问题。
  
  这个问题不容易发现，因为在电脑端即使是chrome的模拟器里都是不存在这种现象的。
  
  
## scroll

iscroll的存在就是因为手机端的滑动没有电脑端流畅。浏览器在我们滚动滚动条放开后，仍会将滚动条的滚动事件延时一段时间，以达到更流畅的效果，但是手机的这个滚动效果明显不那么流畅，演示滚动较短。

如果你试着把页面中可见的一个超过屏幕高度的区域的css设置成

	width: 100%;
	height: 100%;
	position:fixed;
	overflow:scroll;
	
你会发现连延时滚动的效果都没有了，手指离开屏幕时，屏幕就停止滚动了，感觉就像卡在那儿。。。这也是电脑中浏览器不存在的现象。

## touch VS click

之前由于对touch事件使用不当，导致产品经理对我的效果不满意，禁止我用touch，说要的就是click那种停顿感，用户习惯了这种延迟。。。你妹！！！哪有习惯,click的300ms延时只是在手机端出现好不，pc端根本没有延时，哪来的用户习惯。。。好吧，都是我使用不当，才让touch事件蒙冤。

其实使用touch与click对比一下，明显touch带给人更多的流畅感。但是使用得谨慎：

  1、跟click同意的点击事件应绑在touchend上，而不是touchstart或touchmove，否则用户一碰到，手指都还没离开屏幕就触发什么按钮、导航条了，岂不是很奇怪。
  
  2、使用touch事件代替click时，一定要对移动距离做判断，看一下touchstart和touchend发生时的横纵坐标差，做一层判断,就是看用户有没有横向或者纵向滑动。因为touch很奇怪，即使你在A元素上发生touchstart事件，然后手指滑动到B元素上发生touchend事件，仍会触发A元素上的touchend事件。所以如果不做判断，可能就会发生，明明是在滑动页面浏览详情，结果碰到了某个button或tap,你就莫名其妙的提交了表单、跳转了页面。。。

## html5 validation

required直到目前都没有被ios系统支持，目前版本是8.1.3


---
layout: post
title: "使用伪元素解决 retina屏幕下 border 是非retina屏幕2倍宽度问题"
description: "解决 retina屏幕下 border 是非retina屏幕2倍宽度问题"
category: tech
tags: [css]
---
{% include JB/setup %}


经常要给一个元素画框，使用`border-top:1px solid #ddd`的方式，会导致retina屏幕下显示的是2px。这时候可以使用伪元素`:before`结合`content`和`background-color`来实现，这时候是不是retina屏幕就没有关系了。

先看一下效果图：

![image](http://echizen.github.io/assets/blog-img/QQ20150925-1.png)

每个list都是：

	  <li>
	     <div class="item-content">
	         <div class="item-inner">
	             <div class="item-title label">邮箱</div>
	             <div class="item-input">
	                 <input type="email" name="email" placeholder="请填写你的常用邮箱">
	             </div>
	         </div>
	     </div>
	 </li>
	 
css:

	.item-inner:after {
	    content: '';
	    position: absolute;
	    left: 0;
	    bottom: 0;
	    right: auto;
	    top: auto;
	    height: 1px;
	    width: 100%;
	    background-color: #e8e8e8;
	    display: block;
	    z-index: 15;
	    -webkit-transform-origin: 50% 100%;
	    transform-origin: 50% 100%;
	    -webkit-transform: scaleY(.5);
    	transform: scaleY(.5);
	}
	
这种方式其实是利用伪元素创造了一个`height:1px;background-color: #e8e8e8;`元素，视觉上就是一根线。且不会因为retina屏幕发生变化。

**注意：应该判断设备类型，当前只有苹果机有retina的机型，只应对苹果机做出如此的处理。安卓机依旧使用border方式。因为这种方式在大多数低端安卓机上，滚动时线条会抖动，安卓对伪元素的支持效果并不是很好**

小tip:对不同机型做处理可以使用js在入口处做统一判断，然后给`html`加上class，如`ios`,然后给`.ios .item-inner`做出如上处理，这个方式对单页面来说尤为有用，要避免的方式就是在每个页面的js部分判断机型动态添加这个类的傻方式。
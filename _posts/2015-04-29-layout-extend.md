---
layout: post
title: "flexbox 使用注意事项"
description: "flex布局的几个坑，flex的兼容性"
category: tech
tags: [css, basic]
---
{% include JB/setup %}

1. 移动端使用android手机测试效果！！！iphone的支持度很好，跟chrome下模拟器的效果一致。但是android就差的大了。毕竟模拟器只是模拟了屏幕尺寸和头信息。估计是chrome和iPhone的浏览器都用的是webkit内核，效果一致。
2. 为了兼容性，将能加的前缀都加上：


		display: -webkit-box;
		display: -ms-flexbox;
		display: -webkit-flex;
		display: flex;
		
		-webkit-justify-content: space-between;
		justify-content: space-between;
		webkit-box-pack: justify;
		-ms-flex-pack: justify;
		
		-webkit-box-align: center;
		-ms-flex-align: center;
		-webkit-align-items: center;
		align-items: center;
		
		-webkit-flex-wrap: nowrap;
		flex-wrap: nowrap;
		
		-webkit-flex-shrink: 0;
		-ms-flex: 0 0 auto;
		flex-shrink: 0;
		
	![image](http://echizen.github.io/assets/blog-img/QQ20150429-1@2x.png)

	接下来说说我用flex实现这个页面效果所发现的要注意的几点。

3. 使用`flex-shrink = 0 `防止dom塌陷。
	以文件下载那一条为例，这是3栏布局，中间的文件名是多行，而前后的文件标示图和下载链接是有固定大小的，如果不加这个属性会导致当中间的文件名超过一行时会挤压2边的内容，即使你给文件标示图定了固定的大小，还是会被中间的内容压小。所以要给文件标示图和下载链接以下style:
	
		-webkit-flex-shrink: 0;
	    -ms-flex: 0 0 auto;
	    flex-shrink: 0;
	
4. flex下的如果有确定的尺寸的元素，则每个子元素都应该被明确指定flex属性，否则安卓下会破相。
	
		flex: flex-grow flex-shrink flex-basis|auto|initial|inherit;
		
	3中我们给左右2个元素指定了flex的flex-shrink属性，现在为了防止中间的文件名挤压左右元素，要给他flex-grow属性，防止他膨胀，也可以写成：
	
		-webkit-box-flex: 1;
	    -ms-flex: 1;
	    flex: 1; 
	    
5. `flex-wrap`属性安卓4.4以上才支持！！！即使加了前缀。解决方案是给允许换行的子元素`display:inline-block`属性。

	图片展示区域即使在外部容器里加了：
	
		  display: -webkit-box;
		  display: -moz-box;
		  display: -webkit-flex;
		  display: -moz-flex;
		  display: -ms-flexbox;
		  display: flex;
		  -webkit-flex-wrap: wrap;
		  -moz-flex-wrap: wrap;
		  -ms-flex-wrap: wrap;
		  flex-wrap: wrap;
		  
	给子元素`display:inline-block`属性要注意的是，空格和换行符在这里会占位置，譬如你5个子元素，每个宽度20%，这时候会换行！！！因为你的源码里的空格和换行符占了位置。解决方案是给容器加上`font-size: 0px;`，
	
6. display为flex的元素text-overflow: ellipsis失效

	貌似是flex目前还不支持这个。一般解决方案都是曲线救国，在flex的元素里放上子元素，给子元素`block`或`inline-block`的display。
	
	  
###黄金资料

别人整理的bug合集：[https://github.com/philipwalton/flexbugs](https://github.com/philipwalton/flexbugs)

flex虽美，但请慎用！


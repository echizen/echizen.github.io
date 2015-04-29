---
layout: post
title: "flexbox layout"
description: "flex布局，-webkit-box布局"
category: tech
tags: [basic, css]
---
{% include JB/setup %}

##兼容性

用framework7的时候学习了flex布局：

	display: -webkit-box; /* Chrome 4+, Safari 3.1, iOS Safari 3.2+ */
    display: -moz-box; /* Firefox 17- */
    display: -webkit-flex; /* Chrome 21+, Safari 6.1+, iOS Safari 7+, Opera 15/16 */
    display: -moz-flex; /* Firefox 18+ */
    display: -ms-flexbox; /* IE 10 */
    display: flex; /* Chrome 29+, Firefox 22+, IE 11+, Opera 12.1/17/18, Android 4.4+ */

box布局是flex布局的前身，为了兼容性，一般写成上面的格式。在safari和ios系统上必须加`-webkit`前缀。

兼容性：各大系统、浏览器都表现了很好的兼容性

![image](https://echizen.github.io/assets/blog-img/QQ20150426-1@2x.png)


##使用场景

flex是为了更加灵活的布局。flex容器里的元素分配的是相对空间。

1. 有利于屏幕适配。flex布局不依赖于固定尺寸。wrap和justify-content属性让我们在不使用媒体查询的时候也能优雅的适配屏幕布局。
2. 可以让不知道大小的元素在容器里按我们想要的方式布局。这解决了很多因为动态大小元素带来的问题。
3. 可以很好的处理元素之间的空隙问题，可以防止overflow现象。
4. flex布局可以设置元素方向，水平垂直，左向右都可以，然而float是垂直定位，inline是水平定位。
4. 如同float带来的流式布局一样，flex系列属性能完成任意的布局要求，只是相比其他布局的简易而已。所以遇到问题时我们要寻找属性，知道flex什么都可以做。当然我们可以在一个页面中结合多种布局。
5. 但是flex布局对方向改变、元素大小改变、竖屏变横屏这些方面有需求的布局不适合，也不适合复杂的元素嵌套关系的布局。flex布局适合小而美的页面，而栅格布局才适合复杂页面。
6. 不确定大小的元素垂直居中变得可行。


##属性

flex 布局涉及到2方面，父容器（container）和子元素(item)，配合使用才能完成flex布局。(以下图片来源于网络，非原创)

**注意**：`float`, `clear` 和 `vertical-align` 在flex元素上无效.

### container属性

### 1. display

	.container {
	  	display: -webkit-box;
  		display: -moz-box;
  		display: -ms-flexbox;
  		display: -webkit-flex;
  		display: flex;
	}
	

	
### 2. justify-content：单行元素左右空隙分布

	.container {
	  justify-content: flex-start | flex-end | center | space-between | space-around;
	}
	
+ **flex-start (default)**: 一行子元素相对于容器左对齐
+ **flex-end**: 行元素右对齐
+ **center**: 行元素居中
+ **space-between**: 第一个元素不留空隙左对齐，最后一个元素不留空隙右对齐，空隙均匀分布在元素之间
+ **space-around**: 空隙被均匀的分布在元素之间，但由于第一个元素与边界只有左空隙，而元素与元素之间既有前一个元素的右空隙又有后一个元素的左空隙，所以结果是首尾元素距离边界的空隙只有元素与元素之间空隙的一半。

![image](https://echizen.github.io/assets/blog-img/QQ20150426-4@2x.png)

### 3. align-items： 单行元素上下空隙分布

	.container {
	  align-items: flex-start | flex-end | center | baseline | stretch;
	}
	
+ **flex-start**: 子元素与容器上边界对齐
+ **flex-end**: 下对齐
+ **center**: 中心线对其
+ **baseline**: 依据元素内的字下方对齐
+ **stretch** (default): 子元素被拉长至与父容器等高，子元素和父元素上下都对齐。

![image](https://echizen.github.io/assets/blog-img/QQ20150426-5@2x.png)

### 4. align-content： 多行元素上下空隙分布

	.container {
	  align-content: flex-start | flex-end | center | space-between | space-around | stretch;
	}
	
+ **flex-start**: 多行子元素与容器上边界对齐，下方留空隙。
+ **flex-end**: 下对齐。
+ **center**: 居中对齐，上下留白。
+ **space-between**: 多行平均分布，第一行与容器上边界对其，最后一行与容器下边界对其，行于行之间留相同的空隙
+ **space-around**: 多行平均分布，但是上下边界与元素之间也留等高的空隙。

###5. flex-direction：布局方向

	.container {
	  	flex-direction: row | row-reverse | column | column-reverse;
	}
	
![image](https://echizen.github.io/assets/blog-img/QQ20150426-2@2x.png)
	
###6. flex-wrap：子元素换行方式

	.container{
	  flex-wrap: nowrap | wrap | wrap-reverse;
	}
	
+ nowrap (default): 不换行，单行排完 / left to right in ltr; right to left in rtl
+ wrap: 多行，允许换行，从左向右排列 / left to right in ltr; right to left in rtl
+ wrap-reverse: 多行，从右向左排列 / right to left in ltr; left to right in rtl	
![image](https://echizen.github.io/assets/blog-img/QQ20150426-3@2x.png)

###7. flex-flow :flex-direction 和flex-wrap的简洁写法

	flex-flow: <‘flex-direction’> || <‘flex-wrap’>
	
	
	
==============================
	
###item属性

### 1. order：元素排列顺序

	.item {
	  order: <integer>;
	}
	
设置元素排列顺序，这可能在js中动态操作元素时有用。默认是按元素元素书写前后顺序排列。

### 2. flex-grow：相对于其他元素扩大宽度倍数。

	.item {
	  flex-grow: <number>; /* default 0 */
	}

![image](https://echizen.github.io/assets/blog-img/QQ20150426-6@2x.png)

测试地址：

### 3. flex-shrink：相对于其他元素缩小宽度倍数。

	.item {
	  flex-shrink: <number>; /* default 1 */
	}
	
测试地址：[http://www.w3schools.com/cssref/playit.asp?filename=playcss_flex-shrink&preval=1](http://www.w3schools.com/cssref/playit.asp?filename=playcss_flex-shrink&preval=1)

若(代码不完整，仅为说明关键点)：

	<style type="text/css">
	    .item1{
	        flex-shrink:1;
	    }
	    .item2{
	        flex-shrink:2;
	    }
	</style>
	
	<div class="container">
	    <div class="item1"></div>
	    <div class="item2"></div>
	</div>


**注意**：以上示例是把container分成了3份，item1占2/3，item2占1/3。但是实际上浏览器往往计算结果有偏差。（也可能是我理解有偏差，但是琢磨了很久，一直没找到倍数关系）

### 4. flex-basis ：元素初始宽度。

	.item {
	  flex-basis: <length> | auto; /* default auto */
	}
	
### 5. align-self：元素排列靠齐方向。


	.item {
	  align-self: auto | flex-start | flex-end | center | baseline | stretch;
	}
	
属性值含义同align-items。
	
![image](https://echizen.github.io/assets/blog-img/QQ20150426-7@2x.png)


### 6. flex：flex-grow, flex-shrink 和 flex-basis的简写

	.item {
	  flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
	}
	
	
------------------------------
 
 疑惑点：flex-shrink、flex-grow计算。flex-basis为0px和auto的含义
 
 --------------------------
 
 
##黄金参考资料

[https://css-tricks.com/snippets/css/a-guide-to-flexbox/](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

[http://www.w3schools.com/cssref/css3_pr_flex.asp](http://www.w3schools.com/cssref/css3_pr_flex.asp)
w3schools的play it部分改变列子参数很直观。

[http://philipwalton.github.io/solved-by-flexbox/](http://philipwalton.github.io/solved-by-flexbox/)
这是最具体的demo了。


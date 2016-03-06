---
layout: post
title: "触屏事件与鼠标事件"
description: "touch与click，触屏事件与鼠标事件的冲突，移动端开发页面注意事项，移动端click的3000ms延时，touchstart、touchmove、touchend、mouseover、mousemove、mousedown、mouseup、click在移动端的加载顺序；手机端滑动删除效果"
category: tech
tags: [tutorial 进阶 前端 js html5]
---
{% include JB/setup %}

## 故事来源（可忽略）

最近做了一个购物车的页面，既有滑动删除又有点击增减数量，同一区域发生了2件事，发现2个事件没法同时共存。由于这个页面用的是angularjs,开始以为是angularjs与原生js混用，且把原生js没有按angularjs的规则写入angularjs内部，而是写到了angularjs闭合圈的外面,初学angularjs，入门教程都还没看完就在赶项目，不是很熟悉angularjs的事件触发机制，于是跑去看了一天angularjs的事件绑定，以及directive,然后发现不关angularjs的事，测试了一下果然不是。然后又怀疑是事件冒泡的干扰，又去看了几小时事件的冒泡和补获机制，发现也不是。最后才发现是因为手机端touch事件与click事件之间的关系导致。

![image](https://echizen.github.io/assets/blog-img/QQ20141223-1.png)
![image](https://echizen.github.io/assets/blog-img/QQ20141223-2.png)


惨痛教训啊，以后要先试验，再怀疑，再搜索，而不是凭空怀疑导致事件出错的方向，然后google一气。不过这次了解了很多。

## 手机端touchstart、touchmove、touchend、mouseover、mousemove、mousedown、mouseup、click事件加载顺序

### 触屏事件
+ touchstart    触摸开始（手指放在触摸屏上）
+ touchmove     拖动（手指在触摸屏上移动）
+ touchend 触摸结束（手指从触摸屏上移开）
+ touchcancel，是在拖动中断时候触发。

### 鼠标事件
+ mouseover 	鼠标进入
+ mousemove		鼠标移动
+ mousedown		鼠标按下触发
+ mouseup		鼠标抬起触发
+ click			鼠标点击事件，包括mousedown和mouseup2个过程


### 触发规则
在触屏操作后，手指提起的一刹那（即发生touchend后），系统会判断接收到事件的element的内容是否被改变：

+ 如果内容被改变，会解析为touch事件，接下来的click事件都不会触发，
+ 如果内容没有改变，则会解析为click事件，按照mousedown，mouseup，click的顺序触发事件。
特别需要提到的是，在解析为click事件时，只有再触发一个触屏事件时，才会触发上一个事件的mouseout事件。

通常click事件官网文档是说会延时200~300ms.

因此有关于hover的小技巧，当点击过一个按钮之后，这个按钮就会一直处于hover的状态，此时基于这个伪类所设置的css也是起作用的，直到用手指点击另外一个地方，才会完成mouseout事件。

### 触发顺序

测试代码：

	function test_touch(){
	    //isTouchDevice();
	    var firstEmitTime = 0;
	    var aa = document.getElementById("test");
	    var eventTypeArr = ['touchstart', 'touchmove', 'touchend','click', 'mousedown', 'mousemove', 'mouseover', 'mouseup'];
	     
	    for(var k = 0; k < eventTypeArr.length; k++){
	        //利用闭包保存eventType，当回调函数触发时会访问该闭包的环境变量对象,
	        (function(){
	            var eventType = eventTypeArr[k];
	            aa.addEventListener(eventType, function(){
	                var curTime = (new Date()).getTime();
	                if(firstEmitTime === 0){
	                    firstEmitTime = curTime;
	                }
	                //打印当前事件触发时间与第一个事件触发时间的差值
	                var log = eventType + ': ' + (curTime - firstEmitTime);
	                console.log(log);
	            });
	        })();
	    }
	}
	window.onload = test_touch;

正常的轻轻点击一下会触发：

![image](https://echizen.github.io/assets/blog-img/QQ20141223-3.png)

数字表示事件触发间隔，单位是ms，测试环境是chrome浏览器控制台下的Emulation。

当你将激活的模拟器关闭，使用正常的pc网页模式，再点击一下，

![image](https://echizen.github.io/assets/blog-img/QQ20141223-3.png)

+ 移动端，点击一下会触发：

	`touchstart->touchend->mouseover->mousedown-> mouseup->click`;
	
+ 移动端，滑动，触发：
	`touchstart->touchmove->touchend`;
	
+ pc端，点击：
	`mouseover->mousemove->mousedown-> mouseup->click`;
	
+ pc端，移动：
	`mouseover->mousemove`

在真实的手机环境下，我不知道怎样像这样一次性输出这些事件。只能一步步通过alert跟踪，但是要注意的是alert是阻塞事件的，所以不能一次输出，不代表不出发，只能手动在每一步每一次触发一个alert。

关于触发顺序，详细请参考[http://realwall.cn/blog/?p=162](http://realwall.cn/blog/?p=162)

### 关于click事件的300ms延时

这篇文章介绍的很详细：[http://thx.github.io/mobile/300ms-click-delay/](http://thx.github.io/mobile/300ms-click-delay/)。

### 关于我的bug

我根据官方API给元素绑定了滑动事件：

	var startX = 0, startY = 0;  
	var endX = 0,endY = 0;
	var moveArea;
	
	//touchstart事件  
	function touchStartFunc(evt) {  
	    try  
	    {  
	        evt.preventDefault(); //阻止触摸时浏览器的缩放、滚动条滚动等  
	
	        var touch = evt.touches[0]; //获取第一个触点  
	        var x = Number(touch.pageX); //页面触点X坐标  
	        var y = Number(touch.pageY); //页面触点Y坐标  
	        //记录触点初始位置  
	        startX = x;  
	        startY = y;  
	
	    }  
	    catch (e) {  
	        console('touchSatrtFunc：' + e.message);  
	    }  
	}  
	
	//touchmove事件，这个事件无法获取坐标  
	function touchMoveFunc(evt) {  
	    try  
	    {  
	        evt.preventDefault(); //阻止触摸时浏览器的缩放、滚动条滚动等  
	        var touch = evt.touches[0]; //获取第一个触点  
	        var x = Number(touch.pageX); //页面触点X坐标 
	        endX = x; 
	        leftX = x-startX;
	        if(leftX < 0){
	            //var y = Number(touch.pageY); //页面触点Y坐标
	            moveArea = evt.target.parentElement;
	            while(moveArea.getAttribute('move-area')!=('yes')) {
	                moveArea = moveArea.parentElement;
	            } 
	            moveArea.style.marginLeft=+"px";
	        }
	    }  
	    catch (e) {  
	        console('touchMoveFunc：' + e.message);  
	    }  
	}  
	
	//touchend事件  
	function touchEndFunc(evt) { 
	    if(moveArea) { 
	        try {  
	            evt.preventDefault(); //阻止触摸时浏览器的缩放、滚动条滚动等 
	
	            if(endX-startX <= -10){
	                moveArea.style.marginLeft="-112px";
	            }else{
	                moveArea.style.marginLeft="0px";
	            }
	
	            //var text = 'TouchEnd事件触发';  
	            //document.getElementById("result").innerHTML = text;  
	        }  
	        catch (e) {  
	            console('touchEndFunc：' + e.message);  
	        }  
	    }
	}  
	
	//绑定事件  
	function bindEvent() {  
	    var touch_view = document.getElementsByClassName("move_area");
	    for(var i=0;i<touch_view.length;i++){
	        
	            touch_view[i].addEventListener('touchstart', touchStartFunc, false);  
	            touch_view[i].addEventListener('touchmove', touchMoveFunc, false);  
	            touch_view[i].addEventListener('touchend', touchEndFunc, false);  
	        }
	}  
	
	//判断是否支持触摸事件  
	function isTouchDevice() {  
	    try {  
	        document.createEvent("TouchEvent");  
	        console.log("支持TouchEvent事件！");  
	
	        bindEvent(); //绑定事件  
	    }  
	    catch (e) {  
	        console.log("不支持TouchEvent事件！" + e.message);  
	    }  
	}  
	
	window.onload = isTouchDevice; 
	
同时给.move_area中的.nub_opt绑定了angularjs的ng-click也就是click事件。

{% raw %}

	<div class="container-fluid cart_item" ng-repeat='item in items'>
	    <div class="col-xs-12 clear_padding move_area" move-area="yes">
	    <div class="touch_area">
	        <div class="mul_col check_area" style="width:11%" ng-click="checkGoods($index)">
	            <span class="wait_check"  ng-class="{'checked': item.isChecked}"></span>
	        </div>
	        
	        <div class="mul_col goods_thumb " style="width:30%">
	            <img src="{{item.img}}">
	        </div>
	        <div class="mul_col" style="width:59%">
	            <div class="goods_name">
	                {{item.name}}
	            </div>
	            <div class="goods_price_wrap">
	                单价：<span class="goods_price">￥{{item.price}}</span>
	            </div>
	            <div class="goods_nub_wrap">
	                <div class="nub_opt glyphicon glyphicon-plus" ng-click="addNub($index)"></div>
	                <span class="goods_nub">
	                    <input type="number" class="nub_input" value="1" placehover="" value="" ng-model='item.nub'>
	                </span>
	                <div class="nub_opt glyphicon glyphicon-minus" ng-click="minusNub($index)"></div>
	            </div>
	        </div>
	    
	        <div class="delete_btn" ng-click="remove($index)">
	            <img src="__MOBILE_PUBLIC__/Static/images/delete.png">
	        </div>
	    </div>
	</div>
	
{% endraw %}
	
关键是一个很小的问题。。。却导致了很严重的后果。我在touchstart、touchmove、touchend事件绑定的函数中都用了`preventDefault()`。

对于touch事件：**`preventDefault()`不仅会阻止触摸时浏览器的缩放、滚动条滚动等，还会导致鼠标事件被禁止**,我只关注了前一点。。。我的网页已针对移动端适配，在head中已加`<meta name="viewport"
    content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">`，preventDefault()已的禁止缩放滚动已不需要。
    
## 加料区
### 手机端click事件又一特性--点透效果

都说手机端建议使用touch事件，少用touch事件，但是偶尔也会出问题。

典型事例是网页遮罩层绑有touch事件，而遮罩层下有button元素或a标签或某元素绑有click事件，这时当你触发遮罩层的touch事件后，引发的click事件会“穿透”遮罩层，导致下面元素绑有的click事件被触发。

+ 总结一下，点透产生的条件：

	1.A/B两个层上下z轴重叠。

	2.上层的A点击后消失或移开。（这一点很重要）

	3.B元素本身有默认click事件（如a标签和button） 或 B绑定了click事件。
	
+ 点透出现原因
	click事件的300ms延时，当上层元素A的touch触发至click触发的这300ms中，上层元素消失，待click出发时就传递到了它重叠的B元素（不是很明白，事件会通过冒泡或捕获方式传播，为何还能在元素重叠时传播？）
	
+ 解决方案

	1.对于B元素本身没有默认click事件的情况（无a标签等），应统一使用touch事件，统一代码风格，并且由于click事件在移动端的延迟要大很多，不利于用户体验，所以关于触摸事件应尽量使用touch相关事件。

	2.对于B元素本身存在默认click事件的情况,应及时取消A元素的默认点击事件，从而阻止click事件的产生。即应在上例的handle函数中添加代码：`if(eve == "touchend") e.preventDefault();`
	
	3.对于遮盖浮层，由于遮盖浮层的点击即使有小延迟也是没有关系的，反而会有疑似更好的用户体验，所以这种情况，可以针对遮盖浮层自己采用click事件，这样就不会出现点透问题。

参考：[http://www.douban.com/note/430517401/](http://www.douban.com/note/430517401/)

### tap-event的比较规范的写法

[https://github.com/component/tap-event/blob/master/index.js](https://github.com/component/tap-event/blob/master/index.js)

### angularJs事件绑定
angularjs数据双向绑定的强大毋庸置疑，虽说angular的理念是避免频繁操作dom元素，但是在良好交互的需求下，给元素绑事件处理还是很必要的，angualarJs的核心文档中只有ng-click、ng-hide、ng-show区区3个事件。

不建议自己用在angularJs之外再用jquery，事件加载和dom解析会交替产生很多头疼的问题。方案有三：

+ 闭包原生Js中绑定事件
	我这类初学者只能拿这个方案暂时解燃眉之急，没有太大问题产生。要注意angurJs的解析在原生JS之后，也就是`/{/{}}`指令和`ng-repeat`中的内容会在你的JS代码执行之后才解析，这时**如果你要对AngularJs生成的dom元素绑事件，一定要window.load=yourEventFunc。**
	
+ 使用插件
	人笨就要学会多偷懒，angularJs的发展使基于他的插件越来越多，如大名鼎鼎的[angular-bootstrap](http://angular-ui.github.io/bootstrap/)。使用他们封装的事件会事半功倍。
	
+ 自己写directive
	人家写的永远不能满足自己的需求，自己牛逼时，可以自己写directive，当你定义了你的directive后，你就可以通过诸如ng-touchstart来绑定符合angularJs规则的你自己写的事件了。这方面我还没干过，等我自己会写了，再总结。


    

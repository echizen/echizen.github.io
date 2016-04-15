---
layout: post
title: "写个小轮解决繁琐的touch事件"
description: "touch事件的脚手架操作"
category: tech
tags: [js]
---
{% include JB/setup %}

## 背景

对于touch我的内心是抵触的，一方面是因为太灵敏，常导致误操作，另一方面是太繁琐了，为了模仿scroll，要算坐标，进行一群操作。

所以能用浏览器原生scroll的，我是绝对不用touch。

但是最近接了个需求，要做拖拽效果的滚动的导航，不得不上touch家族，先辈们弄了个超级大的组件（2300多行），然后我的需求现有组件实现不了，要改动，改时实在是牵一发动全身，还是自己折腾个吧，不过只能和拒绝多时的touch握手了。

写完之后还想吐槽，touch系列真的是太麻烦了，所以还是抽成一个类，方便以后沿用吧。

# 代码

[戳我，我是代码-touchInit](https://github.com/echizen/js-littleClass/blob/master/touchInit/touchInit.js)

好像也没啥可解释的，就说一下一些收获和注意事项吧。

### touchmove和touchend的绑定时间

一直监听touch事件是很耗性能的。（但是不停的绑事件和删事件也挺耗性能的）。

如果touchstart发生后，touchend发生前,touch过的元素有变更，譬如被删除，想要的滚动效果会发生错乱。

所以一般是在一开始就绑定touchstart事件，但是在touchstart里绑定touchmove和touchend事件，在touchend事件里删除掉touchmove和touchend事件。

	   _onStart: function(event) {

          event = event.originalEvent || event;

          addEventListener(doc.body, moveEvent, this._onMove);
          addEventListener(doc.body, endEvent, this._onEnd);

          this.startCoords = getCoords(event); // for performance , store start coords

        },

        _onMove: function( event ) {
          var distance = {
            x: 0,
            y: 0
          };

          event = event.originalEvent || event;

          // ensure swiping with one touch and not pinching
          if ( event.touches && event.touches.length > 1 || event.scale && event.scale !== 1) return;

          event.preventDefault();

          distance = this._getDistance(event);
          this.coordinates = this._getCoordinate(distance.direction, distance.x, distance.y);
          this._scroll( this.coordinates );
           
          this.onTouchMove(); 
        },

        _onEnd: function( event ) {

          event = event.originalEvent || event;

          this.startCoords = { x: 0, y: 0 };

          this._resetScrollBorder();

          this._scroll( this.scrollBorder );
          
          // do what you want to do
          this.onTouchEnd();

          removeEventListener(doc.body, moveEvent, this._onMove);
          removeEventListener(doc.body, endEvent, this._onEnd);

        }
   
### touch实现滚动的边界处理

因为touch实现滚动的原理，我们会记录每次touch事件结束后的元素左上角坐标值scrollBorder，再在touchmove发生时通过event对象下的坐标值计算和touchstart时的坐标值的差值，将之前的scrollBorder加上这个差值，得出新的元素左上角位置值，再让作用区域通过`translateX`和`translateY`变换成相应的值。每次touchend发生后更新scrollBorder的值。

因为`translateX`和`translateY`可以接收你滚动距离的任何大的值，这就导致没有滚到底的概念，能一直滚下去。所以我在配置中让传递border的边界宽高（默认为窗口screen宽高），在touchmove和touchend中做处理：判断当前坐标是否已经导致目标区域完全滚出了边界区域，如果是，减缓滚动速度，其实就是减少`translateX`或`translateY`的值，我将真实差值/5，再在`touchend`时将元素的scrollBorder重置为区域边界值。

touchmove获取坐标值时对边界的处理：

     _getCoordinate: function(direction, x, y ) {
          var coordinates = {
            x: x,
            y: y
          };

          switch ( direction ) {

            case "right":
              if ( this.scrollBorder.x>=0 ) {
                coordinates.x = Math.round((x - this.scrollBorder.x) / 5 );
                return coordinates;
              }
              break;

            case "left":
              // scroll after right border,divide by 5 is to slow speed
              if ( this.container.offsetWidth - this.wrapWidth <= Math.abs(this.scrollBorder.x) ) {
                coordinates.x = Math.round( -(this.container.offsetWidth - this.wrapWidth) + x / 2 );
                return coordinates;
              }
              break;

            case "down":
              if ( this.scrollBorder.y >= 0 ) {
                coordinates.y = Math.round( (y - this.scrollBorder.y) / 5 );
                return coordinates;
              }
              break;

            case "up":
              if ( this.container.offsetHeight- this.wrapHeight <= Math.abs(this.scrollBorder.y) ) {
                coordinates.y = Math.round( -(this.container.offsetHeight - this.wrapHeight) + y / 5 );
                return coordinates;
              }
            break;
          }

          return {
            x: this.scrollBorder.x + x,
            y: this.scrollBorder.y + y
          };
      }

touchend对边界的处理：

    _resetScrollBorder: function(){
          var coordinates = this.coordinates;

          if(coordinates.x>0){
            this.scrollBorder.x = 0;
          }else if (-coordinates.x >= this.container.offsetWidth - this.wrapWidth){
            this.scrollBorder.x = -(this.container.offsetWidth - this.wrapWidth);
          }else{
            this.scrollBorder.x = this.coordinates.x;
          }

          if(coordinates.y>0){
            this.scrollBorder.y = 0;
          }else if (-coordinates.y >= this.container.offsetHeight - this.wrapHeight){
            this.scrollBorder.y = -(this.container.offsetHeight - this.wrapHeight);
          }else{
            this.scrollBorder.y = this.coordinates.y;
          }
          
       }
 
### touch存在时导致的click点透

   我遇到2种场景，一种是用touch做拖拽效果的滚动会有动画效果的延时，只要保证动画设置时间不小于300ms就不会影响click。
   一种是用touch做自然滚动时，这时滚动没有延时很快，所以我去掉了click事件，在touch中判断又没有发生touchMove并且move的值是否超过一定值来决定是执行touch的滚动效果，还是click效果。
   
touchMove中：

 		// 滚动或者点击的断定
          if(this.config.direction == 'horizontal'){
            this.ismove = Math.abs(distance.x) > 10?true:false;
          }else{
            this.ismove = Math.abs(distance.y) > 10?true:false;
          } 
          
touchEnd中：

		  if(this.ismove){
            this._resetScrollBorder();
            this._scroll( this.scrollBorder );
            this.onTouchEnd();
          }else{
            this.onClick(event.target);
          }
          this.ismove = false;



### this的绑定

是否原型链里处处`bind(this)`显得很繁琐呢，不如把这些操作都扔到一个函数里吧，然后在构造器中调用一下。


	  function proxy( fn, context ) {
	
	    return function() {
	      return fn.apply( context, Array.prototype.slice.call(arguments) );
	    };
	
	  }
	  
其实在调用时直接`call(this)`也可以。
	  
### 扩展对象

如果没有jquery，改怎么替代`$.extend`呢？

	  function extend( destination, source ) {
	
	    var property;
	
	    for ( property in source ) {
	      destination[property] = source[property];
	    }
	
	    return destination;
	
	  }
	  
### 为了兼容性，做好降级

譬如是否支持touch，是否支持动画。

	var doc = document,
      win = window;
      
     var isTouch = 'ontouchstart' in win,
	  startEvent = isTouch ? 'touchstart' : 'mousedown',
	  moveEvent = isTouch ? 'touchmove' : 'mousemove',
	  endEvent = isTouch ? 'touchend' : 'mouseup',
	  
根据判断结果选择不同的处理。
  
  	this._scroll = supportTransform ? this._scrollWithTransform : 	this._scrollWithoutTransform;
  
	  function getCoords(event) {
	    // touch move and touch end have different touch data
	    var touches = event.touches,
	        data = touches && touches.length ? touches : event.changedTouches;
	
	    return {
	      x: isTouch ? data[0].pageX : event.pageX,
	      y: isTouch ? data[0].pageY : event.pageY
	    };
	  }
   
     
### 尾声
  
  好水啊,写不下去了...就是个简易的touch事件通用化操作，需要时拉过去用就好。宝宝再也不怕touch模拟滚动了


然后会把拖拽效果的滚动条代码和踩坑经历也放出来，地址占坑：[https://github.com/echizen/dragNav](https://github.com/echizen/dragNav)
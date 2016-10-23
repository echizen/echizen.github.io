---
layout: post
title: "animation动画初步"
description: "animation动画初步，动画性能"
category: tech
tags: [css,性能]
---
{% include JB/setup %}


小白初次做动画，涨了一批基础知识。拿到的需求稿的效果就是跟ppt一样，文字嗖嗖嗖的淡入、平移、左侧渐出、右侧渐出等这些基础效果。

## 元素分析

动画不是刷一下都蹦出来的，是要根据行为方式和出现时间拆分一个个动画元素。看到的可能是各种物体，实际都可以做成一个个div元素，通过background-image来展现多种样子。

## 动起来：animation + keyframes

例如一个文字从上至下滑落淡出的效果：

    keyframes fadeInDown{
        0% {
            transform: translate3d(0, -1.6rem, 0) scale(0,0);
            opacity: 0;
        }
        100% {
            transform: translate3d(0, 0, 0) scale(1,1);
            opacity: 1;
        }
    }
    
    animation: fadeInDown 0.4s linear 0.2s forwards;
    
整个动画的基本效果都可以用animation调出来，第一个参数指定动效，具体效果可以在keyframes中定义；第二个参数是执行时间；第三个参数是动画效果函数，线性还是渐入渐出等；第四个参数很关键，表示动画开始执行的时间，正是这个参数让我们能让一系列元素按照设定的顺序进行动画播放；第5个参数是指定动画完成后停留的状态，譬如forwards指定保持结束时的状态，默认是保持开始时的状态。

## 注意点

### 性能
由于transform和opacity可以避免触发layout，而是使用javascript - style - composite的流程渲染，且会创建新的渲染层，每个渲染层的纹理都是上传到GPU处理，性能更好。

所以一般对于以下常见的动画效果，可以用这些方式优化性能：

1. 位移：不要使用`left`、`top`的改变来实现，而是使用`transform: translate3d(-0.2rem, -1.6rem, 0)`来实现，或者不使用`translate3d`,使用`transform: translateX(-0.2rem);transform: translateY(-1.6rem)`的方式。
2. 大小：不要直接改变`width`、`height`，而是使用`transform:scale(0.8,0.8)`这种方式。
3. 显隐：不要使用`display:none`、`display:block`，而是使用`opacity:0`、`opacity:1`。

这个性能的提升不仅仅是从chrome的timeline上的曲线得知，到了低端一点的安卓机上会有明显感受，使用前一种方式的会有卡顿感，使用后一种方式会感觉到明显的流畅了。卡顿的强烈与否取决于动画的复杂度。

### 渲染时间

如果你的动画元素使用了背景图，在图片没加载完成前，你的动画都不能正常渲染，而是等到图片加载完成后，之前的动画刷的一下全蹦出来了。

所以需要确保所依赖的图片加载完成后再进行动画。

这是后可能需要js来起点作用了，现将有动画效果的class放到`.animation`下。暴力一点的办法是在dom中直接添加`display:none`的<img />加载出需要的图片，然后在图片上绑上onload事件来触发动画执行的js。当然有可能img都onload了，你的js还没加载或解析完，所以需要双向通知，譬如：

    <script type="text/javascript">
        function imgReady(){
            if(typeof window.__initAnimation === "function"){
                window.___initAnimation();
            }else{
                window.__imgLoading = true;  
            }
        }
    </script>
	<img class="pre-img" src="http://s16.mogucdn.com/p2/161016/upload_74h3i068d0e8f345a2fjdldg7aa8k_750x621.png" onload="imgReady()"/>
	
js中：

        // 如果img已经加载好，直接执行动画
		if(window.__imgLoading === true){
			initAnnimation();
		}else{
			// 否则注入动画执行函数，由img得onload调用
			window.__initAnimation = initAnnimation;
		}
		
		function initAnnimation(){
			$ele.addClass('animation');
		}
		
### 兼容性

低端机上不出现动画，低端机怎么判断呢？一般不支持几个关键css动画属性的机器我们就不出现动画了，譬如我这里是transform、animation，可以在动画执行前进行检查当前机器的div的style下有没有这些属性：

	// 检查兼容性
	var supports = (function() {
       var div = document.createElement('div'),
           vendors = 'Khtml Ms O Moz Webkit'.split(' '),
           len = vendors.length;

       return function( prop ) {
          if ( prop in div.style ) return true;

          prop = prop.replace(/^[a-z]/, function(val) {
             return val.toUpperCase();
          });
          var i = len;
          while( i-- ) {
             if ( vendors[i] + prop in div.style ) {
                return true;
             }
          }
          return false;
       };
    })();

    var supportTransform = supports('transform');
    var supportAnimation = supports('animation');
    if(!supportTransform || !supportAnimation){
		$ele.remove();
		return;
	}

这些都是很基础的动画，做过了简单总结下。深入的动画不是我的方向啦。





---
layout: post
title: "chrome timeline的简单使用"
description: "chrome timeline的简单使用"
category: tech
tags: [tool]
---
{% include JB/setup %}

非常遗憾，折腾了一个双休日的chrome timeline,还是没玩溜，不想再继续研究了，先把所了解的记录下来。其实在chrome控制台那么小的区域里展示了那么多张图标还是让人眼花缭乱的，而且事件进度条层层叠叠也不是很直观，在一个比较大的文件里还是挺难定位到性能问题究竟出在那些脚本上。

（附：本来想用测试一下字符串的拼接方式谁更高效，1）字符串直接用连接符『+』连接。2）使用数组`[].join('')`的方式。循环拼接了一段长长的dom字符串，大家timeline后发现只有scrpting的时间区别，例子不具有代表性。不过结论是，chrome下，方式1比方式2确实更高效，方式1是40多毫秒，方式2是60多毫秒）

基本用法可以查看谷歌开发文档。[https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/timeline-tool?hl=en](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/timeline-tool?hl=en)

**友情提醒：**

**1. 在启用timeline之前做好使用隐身模式并保证所以扩展程序和插件不可用，避免受干扰。**

**2. 每次timeline的结果数据稍有差别，如果需要准确点的值，需要多观察几次求平均值**


# overview panel - 找到有问题的帧

![image](https://echizen.github.io/assets/blog-img/20160320-1.png)

这个panel包含的信息有帧信息（FPS）、内存信息（CPU）、网络信息（NET）。

注意FPS有红色长条的区域，那是有性能问题的地方，你可以逐一分析每个红色长条提示的性能问题，点击该区域，拖拽激活区域以缩放或扩大，将性能分析定位到这个红色长条上，然后再在底下的面板中查看详细信息。

CPU我还不会看如何体现是有性能问题的地方。。。net一栏可以从network那边获得更直观详细的信息。

# Flame Chart panel - 定位到导致性能问题的调用过程

CPU 堆栈调用追踪区域。

![image](https://echizen.github.io/assets/blog-img/20160320-2.png)

最上面一行是每个任务集合执行所消耗的时间，如果某个集合消耗的的时间过长（>16.6ms）导致性能问题，左上角会有红色三角形警告，summary也会提示帧长度过长导致页面卡顿。

下方还有n多平行的长条，有性能问题的事件右上角依旧有红色三角形，具体的问题提示可以在summary看到。

譬如我打开的这个页面的性能问题是每次都发生了recalculate style，然后layout，且重复多次。

![image](https://echizen.github.io/assets/blog-img/20160320-3.png)
![image](https://echizen.github.io/assets/blog-img/20160320-4.png)

# Details panel

detail面板针对不同的事件有不同的tab，一般都有summary、Bottom-Up、call-tree、Event-Log。如果你点击的是Flame Chart每帧所耗时间的区域条，面板里还会出现layers选项。

## summary - 饼形图展示各阶段耗时情况

展示了你所选中的区域各部分工作（loading、scripting、rendering、painting、other、idle中的几项）消耗时间的饼形图。

对于js函数，他还能显示函数调用链方便定位有问题的脚本行数，如上面那张layer阶段导致重绘的call stacks:

![image](https://echizen.github.io/assets/blog-img/20160320-5.png)

## Bottom-Up - 找到最耗时的时间

主要是时间分析，每个渲染阶段耗时，每个事件的耗时。

![image](https://echizen.github.io/assets/blog-img/20160320-6.png)

## call-tree

详细列出了每个事件调用链中各个事件的耗时情况。

## event-log

展示所有阶段包括loading、javascripting、rendering、painting中各事件的耗时情况，并提供了filter输入框和按钮供你快速过滤查找你感兴趣的东西。

## layers

比较有趣的layers，你可以看到真实的没帧执行后浏览器页面的样子，这是个实时截图，虽然调试时你能看到js执行顺序，但是你看不到浏览器运行你的js指令和自己的指令和整合的效果。这里可以让你感受一下，你可以一帧帧的点进去看，也能非常直观的反映出渲染慢在哪个区域。

![image](https://echizen.github.io/assets/blog-img/20160320-7.png)

# console rendering panel

EST键调起console面板，点击面板左侧竖的"..."选出展示rendering面板，然后打开rendering面板，可以看到可展示的选项有：

1. Enable paint flashing：开启显示重绘区域的功能，重绘区域将以绿色高亮显示（如下图中倒计时区域就在不停的随着刷新重绘）
2. show layer borders：显示渲染层，以矩形框显示了每个渲染层的边界
3. show FPS meter：动态展示及时的帧数
4. show scrolling perf issues：能够显示滚动时监听的的事件类型
5. emulate print media

![image](https://echizen.github.io/assets/blog-img/20160320-8.png)

刚刚逮到个鲜活的案例，有个抽奖的动画效果，在iphone6p ios 8.4 下频繁导致crash，而且一打开页面就会crash。

看代码：

	// 『点击换新装』按钮的动画
	.change_new_clothing .cnc_btn img {
	  width: 100%;
	  height: auto ;
	  -webkit-animation: btnzoom 1s linear infinite alternate ;
	          animation: btnzoom 1s linear infinite alternate ;
	}
	@-webkit-keyframes btnzoom {
	  from {
	    -webkit-transform: scale3d(1, 1, 1);
	            transform: scale3d(1, 1, 1);
	  }
	  to {
	    -webkit-transform: scale3d(0.9, 0.9, 1);
	            transform: scale3d(0.9, 0.9, 1);
	  }
	}
	@keyframes btnzoom {
	  from {
	    -webkit-transform: scale3d(1, 1, 1);
	            transform: scale3d(1, 1, 1);
	  }
	  to {
	    -webkit-transform: scale3d(0.9, 0.9, 1);
	            transform: scale3d(0.9, 0.9, 1);
	  }
	}
	
	// 『换新装』图片的动画
	
	.change_new_clothing .cnc_clothing .new_clothing {
	  -webkit-transform: rotateZ(-180deg);
	      -ms-transform: rotateZ(-180deg);
	          transform: rotateZ(-180deg);
	}
	.change_new_clothing .cnc_clothing .move-out {
	  -webkit-animation: rotationout 1s forwards ;
	          animation: rotationout 1s forwards ;
	  -webkit-transform-origin: bottom ;
	      -ms-transform-origin: bottom ;
	          transform-origin: bottom ;
	}
	@-webkit-keyframes rotationout {
	  from {
	    -webkit-transform: rotateZ(0deg);
	            transform: rotateZ(0deg);
	    display: block ;
	  }
	  to {
	    -webkit-transform: rotateZ(-180deg);
	            transform: rotateZ(-180deg);
	    display: block ;
	  }
	}
	@keyframes rotationout {
	  from {
	    -webkit-transform: rotateZ(0deg);
	            transform: rotateZ(0deg);
	    display: block ;
	  }
	  to {
	    -webkit-transform: rotateZ(-180deg);
	            transform: rotateZ(-180deg);
	    display: block ;
	  }
	}
	.MCUBE_MOD_ID_7084 .change_new_clothing .cnc_clothing .move-in {
	  -webkit-animation: rotationin 1s forwards ;
	          animation: rotationin 1s forwards ;
	  -webkit-transform-origin: bottom ;
	      -ms-transform-origin: bottom ;
	          transform-origin: bottom ;
	}
	@-webkit-keyframes rotationin {
	  from {
	    -webkit-transform: rotateZ(180deg);
	            transform: rotateZ(180deg);
	    display: block ;
	  }
	  to {
	    -webkit-transform: rotateZ(0deg);
	            transform: rotateZ(0deg);
	    display: block ;
	  }
	}
	@keyframes rotationin {
	  from {
	    -webkit-transform: rotateZ(180deg);
	            transform: rotateZ(180deg);
	    display: block ;
	  }
	  to {
	    -webkit-transform: rotateZ(0deg);
	            transform: rotateZ(0deg);
	    display: block ;
	  }
	}
	
光看代码会发现`transform`中加了很多`rotateZ`，这样每一个都会创建一个渲染层，会导致cpu增长。`scale3d`调用了GPU硬件加速，配合`animation`也很消耗性能。其实在现代浏览器中都没问题，但是在iphone6plus 8.3中还是会导致crash，去掉其中一个dom的动画效果，都还是会crash。可能iphone6p 屏幕大GPU、CPU消耗的更厉害，而8.3版本太低，软硬件不是很匹配，webview中的内存管理的不是很好，便导致了这个crash的发生。

![image](https://echizen.github.io/assets/blog-img/20160320-9.png)

截了张rendering的图，可以看出帧数饱和频率很高，持续在60fps，导致动画会有卡顿。『点击换新装』按钮独自占据着一个渲染层。

![image](https://echizen.github.io/assets/blog-img/20160321-11.png)

也截了张页面加载时timeline的图，1000ms-1800ms之间，CPU陡增后居高不下，1800ms之后fps稳定在一个很高的值，这都是性能不太好的表现。

![image](https://echizen.github.io/assets/blog-img/20160320-10.png)

当我们把按钮动画效果除去，把所有`transform`中的`rotateZ`转换为`rotate`以避免创建新的渲染层，之后，fps下降了很多，也没有再出现crash。（但是我并没有发现CPU和GPU下降，不知道怎么解释）。最终解决方案就是针对版本降级动画方案。

# something else

我是因为好奇才研究了这些功能，但是在日常大多数情况下，通过修改脚本对程序性能的改善不大。现代浏览器内存管理优化都很好，性能都很好足以撑住；浏览器引擎自己对我们的脚本会有优化，我们看到的脚本和他真正执行的脚本并不是完全一样，引擎会自己转化为性能好的脚本；脚本再耗时，也比不上网络io的耗时，两者差的是百万级别，所以优化网站第一步还是尽量用文件压缩合并等来减少网络请求和加速数据返回时间。优化网络请求带来的成效更大。

但是毕竟移动端和老的ie浏览器没那么强大，所以还是会出现性能问题，当我们的页面出现问题时，能够通过chrome timeline系列工具发现问题所在或者提前检测问题发生的可能性，不至于出了问题后手足无措。

## 疑惑

CPU状况不会看，引入了jquery等大型库后，函数调用连太长，不能很快定位到出问题的脚本在那里。

# 黄金外链

[How to Look at Performance](https://developers.google.com/web/tools/chrome-devtools/profile/evaluate-performance/timeline-tool?hl=en)



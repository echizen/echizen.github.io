---
layout: post
title: "从浏览器渲染看页面性能优化"
description: "learn page performance, use chrome timeline，页面性能，从浏览器渲染看页面性能优化"
category: tech
tags: [performance]
---
{% include JB/setup %}

经常看到高性能的js该怎么怎么写，各种建议，却不知道如何证实这些说法，如何观测自己的js,css是否高效。遇到页面渲染缓慢（其实现代的chrome下也感觉不到慢，但是ie和手机上对于页面渲染速度就有感受了），也不知道问题出在哪一环节。于是这周看了一下chrome 的timeline使用，顺便了解了一些与性能相关的东西。

# 性能感知-RAIL模型

用户对页面性能的感知来源于四方面：

1. 对于用户操作的响应
2. 动画效果
3. 闲置时间
4. 加载速度

![image](https://echizen.github.io/assets/blog-img/rail.png)

## 时间控制

为了让用户体验良好，对于一些时间节点需要了解。


| Delay	User | Reaction |
| ------ | ----: |
| 0 - 16ms | 屏幕更新每帧动画的时间（1000/60），不可能再16ms以内更新屏幕显示内容 |
| 0 - 100ms | 用户操作响应敏感时间，必须在100ms以内给用户的操作以反馈，才能让用户感觉流畅 |
| 300 - 1000ms | 加载页面或改变视图的可接受时间 |

## 原则

### response: 少于100ms

### Animation: 每16ms更新frames

页面渲染过程包括：javascript、style、layout、paint、composite

![image](https://echizen.github.io/assets/blog-img/render-frame.png)

1. 触发JavaScript。一般来说，我们会使用JavaScript来实现一些视觉变化的效果。比如用jQuery的animate函数做一个动画、对一个数据集进行排序、或者往页面里添加一些DOM元素等。当然，除了JavaScript，还有其他一些常用方法也可以实现视觉变化效果，比如：CSS Animations, Transitions和Web Animation API。
2. 计算样式。这个过程是根据CSS选择器，比如.headline或.nav > .nav_item，对每个DOM元素匹配对应的CSS样式。这一步结束之后，就确定了每个DOM元素上该应用什么CSS样式规则。
3. 布局。上一步确定了每个DOM元素的样式规则，这一步就是具体计算每个DOM元素最终在屏幕上显示的大小和位置。web页面中元素的布局是相对的，因此一个元素的布局发生变化，会联动地引发其他元素的布局发生变化。比如，<body>元素的宽度的变化会影响其子元素的宽度，其子元素宽度的变化也会继续对其孙子元素产生影响。因此对于浏览器来说，布局过程是经常发生的。
4. 绘制。本质上就是填充像素的过程。包括绘制文字、颜色、图像、边框和阴影等，也就是一个DOM元素所有的可视效果。一般来说，这个绘制过程是在多个层上完成的。
5. 渲染层合并。由上一步可知，对页面中DOM元素的绘制是在多个层上进行的。在每个层上完成绘制过程之后，浏览器会将所有层按照合理的顺序合并成一个图层，然后显示在屏幕上。对于有位置重叠的元素的页面，这个过程尤其重要，因为一旦图层的合并顺序出错，将会导致元素显示异常。

每一帧动画都要经历以上5个过程。

动画并不仅限与js和css动画，浏览器原生的`scroll`和`touchmoves`事件都会导致浏览器动画渲染。

浏览器对每一帧画面的渲染工作需要在16毫秒（1秒 / 60 = 16.66毫秒）之内完成，由于浏览器在每帧动画执行期间都需要一些时间来执行自己的一些操作（比如渲染队列的管理，渲染线程与其他线程之间的切换等等），因此单纯的渲染工作要控制在10ms以内。


### idle：好好利用

可以利用空闲时间进行网络请求、异步操作。优化好的页面首屏数据都是很小的，接下来的数据都是异步获取后渲染。

良好交互体验的做法还要控制每个异步操作在50ms以内完成，然后将优先处理用户操作应有的反馈。

### Load：数据传输要少于1000ms

如果有数据请求、接口响应的时间超过这个，需要告诉用户，譬如转个菊花告诉用户『正在加载中』。

# 渲染性能

## 渲染模式

虽然理论上每一帧的渲染都要经历以上5个过程，但是对于我们js触发改变的css属性不同，其实这个过程并不是都会发生的。

### 1.javascript - style - layout - paint - composite

如果修改一个DOM元素的”layout”属性，也就是改变了元素的样式（比如宽度、高度或者位置等），那么浏览器会检查哪些元素需要重新布局，然后对页面激发一个reflow(重绘)过程完成重新布局。被reflow的元素，接下来也会激发绘制过程，最后激发渲染层合并过程，生成最后的画面。

### 2.javascript - style - paint - composite

如果修改一个DOM元素的“paint only”属性，比如背景图片、文字颜色或阴影等，这些属性不会影响页面的布局，因此浏览器会在完成样式计算之后，跳过布局过程，只做绘制和渲染层合并过程。

### 3.javascript - style - composite

如果你修改一个非样式且非绘制的CSS属性，那么浏览器会在完成样式计算之后，跳过布局和绘制的过程，直接做渲染层合并。

第三种方式在性能上是最理想的，对于动画和滚动这种负荷很重的渲染，我们要争取使用第三种渲染流程。

## javascript:提升执行效率

页面里的动画效果大多是通过JavaScript触发的。有些是直接修改DOM元素样式属性而产生的，有些则是由数据计算而产生的，比如搜索或排序。错误的执行时机和太长的时间消耗，是常见的导致JavaScript性能低下的原因。

### 使用requestAnimationFrame

如果你想在动画刚刚发生的那一刻就运行一段JavaScript。那么能保证这个运行时机的，就是`requestAnimationFrame`。

	//
	function updateScreen(time) {
	  // 你要干的
	}
	
	requestAnimationFrame(updateScreen);
	
如果你的js代码是要改变样式，制作动画效果，那么`requestAnimationFrame`能保证你的代码在浏览器下一帧中触发，让你的动画流畅起来。

### 耗时长的JavaScript代码使用Web Workers

JavaScript代码是运行在浏览器的主线程上的。与此同时，浏览器的主线程还负责样式计算、布局，甚至绘制（多数情况下）的工作。可以想象，如果JavaScript代码运行时间过长，就会阻塞主线程上其他的渲染工作，很可能就会导致帧丢失。

因此，你需要认真规划一下你的JavaScript程序的运行时机和运行耗时。比如，如果你要在一个动画（比如页面滚动）执行过程中运行JavaScript程序，那么理想情况是把这段JavaScript程序的运行耗时控制在3-4毫秒以内。如果长于这个时间，那么就有帧丢失的风险。另一方面，在浏览器空闲的时候，你可以有更多时间来运行JavaScript程序。

大多数情况下，你可以把纯计算工作放到Web Workers中做（如果这些计算工作不会涉及DOM元素的存取）。一般来说，JavaScript中的数据处理工作，比如排序或搜索，一般都适合这种处理方式。

	var dataSortWorker = new Worker("sort-worker.js");
	dataSortWorker.postMesssage(dataToSort);
	
	// 现在主线程是空闲的可以用来干其他事
	
	dataSortWorker.addEventListener('message', function(e) {
	   var sortedData = e.data;
	   // 一些操作，如更新页面中的数据
	});
	
### 划分DOM更新，再多个frame中完成

由于Web Workers无法访问DOM元素。如果你的JavaScript代码需要存取DOM元素，也就是说必须在主线程上运行，那么可以考虑批处理的方式：把任务细分为若干个小任务，每个小任务耗时很少，各自放在一个requestAnimationFrame中回调运行。

	var taskList = breakBigTaskIntoMicroTasks(monsterTaskList);
	requestAnimationFrame(processTaskList);
	
	function processTaskList(taskStartTime) {
	  var taskFinishTime;
	
	  do {
	   	var nextTask = taskList.pop();
	
	    processTask(nextTask);
	
	    // 获取本次任务的执行结束时间
	    taskFinishTime = window.performance.now();
	    // 如果这一帧中任务执行超过3ms，就将任务队列中的任务扔到下一帧中
	  } while (taskFinishTime - taskStartTime < 3);
	
	  if (taskList.length > 0)
	    requestAnimationFrame(processTaskList);
	
	}
	
## layout：尽可能避免触发

对于DOM元素的“几何属性”的修改，比如width/height/left/top等，都需要重新计算布局。几乎所有的layout都是在整个文档范围内发生的。 如果页面中含有很多元素，那么计算这些元素的位置和维度的工作将耗费很长时间。

### 使用flexbox布局

如果必须修改元素的位置和尺寸属性，必须触发layout，而设备允许的情况下，使用flexbox布局能改善性能。

但是在任何情况下，不管是是否使用Flexbox，你都应该努力避免同时触发所有布局，特别在页面对性能敏感的时候（比如执行动画效果或页面滚动时）。

### 减少需要执行样式计算的元素的个数

在过去，如果你修改了body元素的class属性，那么页面里所有元素都要重新计算样式。幸运的是，在某些现代的浏览器中不再这样做了。他们会对每个DOM元素维护一个独有的样式规则小集合，如果这个集合发生改变，才重新计算该元素的样式。也就是说，某个元素样式的改变不一定会导致对其他所有元素重新计算样式，得看这个元素在DOM树中的位置、具体是什么样式发生改变。

虽然现代浏览器没那么傻了，样式计算是直接对那些目标元素执行，而不是对整个页面执行。但是如果你考虑兼容性要照顾上一辈浏览器，还是尽量最小区域内改变样式。

以BEM (Block, Element, Modifier)的方式编写CSS代码，能达到最好的样式计算的性能。然而在没有web Components化由多人协同开发的项目中很容易导致样式冲突。

### 避免强制同步布局事件

在JavaScript脚本运行的时候，它能获取到的元素样式属性值都是上一帧画面的，都是旧的值。因此，如果你想在这一帧开始的时候，读取一个元素box的height属性，你可以会写出这样的JavaScript代码：

	requestAnimationFrame(changeBox);
	function changeBox() {
	  var boxOffsetHeight = box.offsetHeight;
	  var boxColor = box.style.color;
	}

但是如果你一边改样式，一边读样式就会很耗性能了。

	requestAnimationFrame(changeBox);
	function changeBox() {
	  var boxOffsetHeight = box.offsetHeight;
	  box.offsetHeight = 3000;
	  var boxColor = box.style.color;
	  box.style.color = '#333';
	}
	
这样浏览器每次都得先读再写，又读再写，每一次写都是一次重绘，发生了2次重绘太耗性能。我们应该先读后写，读出上一帧的值，再改变样式。

如果这种读写混搭风发生再循环里就更可怕了，譬如说这样：

	for (var i = 0; i < length; i++) {
	  container[i].style.height = box.height + 'px';
	}
	  
这样每次循环，都得先读取box的高度值再去改变对应container的高度，这个过程每次都必须发生重绘让上一次的`style.height`的样式改变生效，以保证取到的`box.height`是及时性高的准确值，然而我们如果对container的改变不影响box，其实这个过程就是没必要的。可以先缓存起box的值：

	var boxHeight = box.height;
	for (var i = 0; i < length; i++) {
	  container[i].style.height = boxHeight + 'px';
	}
	
## paint:简化复杂度、减小区域

如果layout被触发，那么接下来paint一定会被触发。因为改变一个元素的几何属性就意味着该元素的所有像素都需要重新渲染。除此之外，如果改变元素的非几何属性，也可能触发paint，比如背景、文字颜色或者阴影效果，尽管这些属性的改变不会触发layout。

绘制并非总是在内存中的单层画面里完成的。实际上，浏览器在必要时将会把一帧画面绘制成多层画面，然后将这若干层画面合并成一张图片显示到屏幕上。

这种绘制方式的好处是，使用tranforms来实现移动效果的元素将会被正常绘制，同时不会触发对其他元素的绘制。这种处理方式和思想跟图像处理软件（比如Sketch/GIMP/Photoshop）是一致的，它们都是可以在图像中的某个单个图层上做操作，最后合并所有图层得到最终的图像。

在页面中创建一个新的渲染层的最好方式就是使用CSS属性will-change，Chrome/Opera/Firefox都支持该属性。同时再与transform属性一起使用，就会创建一个新的组合层。对于那些目前还不支持will-change属性、但支持创建渲染层的浏览器，比如Safari和Mobile Safari，你可以使用一个3D transform属性来强制浏览器创建一个新的渲染层。

但是每创建一个新的渲染层，就意味着新的内存分配和更复杂的层的管理。所以创建前后应该使用chrome DevTools来分析其实际性能表现，以确认是否创建。

浏览器会把两个相邻区域的渲染任务合并在一起进行，这将导致整个屏幕区域都会被绘制。在DPI较高的屏幕上，固定定位的元素会自动地被提升到一个它自有的渲染层中。但在DPI较低的设备上却并非如此，因为这个渲染层的提升会使得字体渲染方式由子像素变为灰阶，我们需要手动实现渲染层的提升。减少绘制区域通常需要对动画效果进行精密设计，以保证各自的绘制区域之间不会有太多重叠，或者想办法避免对页面中某些区域执行动画效果。

对于各种样式效果，各种css属性导致的绘制复杂度是不一样的。比如，background: red就比box-shadow: 0, 4px, 4px, rgba(0,0,0,0.5)更复杂，所以在实现一个效果时，我们需要思考是否有其它方式来达到同样效果。

**paint是整个渲染流水线中耗时最长的一环，因此也是最需要避免发生的一环。特别是在动画效果中。每帧10毫秒的时间预算一般来说是不足以完成绘制工作的，尤其是在移动设备上**


### composite:渲染层合并,控制层数量

我们可以优化的是避免触发layout，而是使用第三种类型的渲染（`javascript - style - composite`）。

css属性中能够直接使用第三种类型渲染模式的只有`opacity`和 `transform`。

[https://csstriggers.com/](https://csstriggers.com/)：css属性是否能触发布局、绘制或渲染层合并的清单。

对于要进行动画操作的元素：

	.moving-element {
	  will-change: transform;
	}

或者，对于旧版本或不支持will-change属性的浏览器：

	.moving-element {
	  transform: translateZ(0);
	}

使用这个CSS属性能提前告知浏览器：这个元素将会执行动画效果。从而浏览器可以提前做一些准备，比如为这个元素创建一个新的渲染层。

但是，**应用了transforms/opacity属性的元素必须_独占一个渲染层**。

创建一个新的渲染层并不是免费的，它得消耗额外的内存和管理资源。每个渲染层的纹理都需要上传到GPU处理，因此使用它们时，我们还需要考虑CPU和GPU之间的带宽问题、以及有多大内存供GPU处理这些纹理的问题。

所以在使用渲染层合并属性时，也要控制层数量。

## 用户操作响应防抖动

在理想情况下，当用户在设备屏幕上触摸了页面上某个位置时，页面的渲染层合并线程将接收到这个触摸事件并作出响应，比如移动页面元素。这个响应过程是不需要浏览器主线程的参与的，也就是说，不会导致JavaScript、布局和绘制过程的发生，不会导致卡顿。

但是，如果你对这个被触摸的元素绑定了输入事件处理函数，比如touchstart、touchmove或者touchend，那么渲染层合并线程必须等待这些被绑定的处理函数执行完毕之后才能被执行。因为你可能在这些处理函数中调用了类似preventDefault()的函数，这将会阻止输入事件（touch/scroll等）的默认处理函数的运行。事实上，即便你没有在事件处理函数中调用preventDefault()，渲染层合并线程也依然会等待，也就是用户的滚动页面操作被阻塞了，表现出的行为就是滚动出现延迟或者卡顿（帧丢失）。

![image](https://echizen.github.io/assets/blog-img/ontouchmove.jpg)

所以基于这种机制，你必须确保对用户输入事件绑定的任何处理函数都能够快速执行完毕，以便腾出时间来让渲染层合并线程来完成它的工作。

输入事件处理函数，比如scroll/touch事件的处理，都会在requestAnimationFrame之前被调用执行。因此，如果你在上述输入事件的处理函数中做了修改样式属性的操作，那么这些操作会被浏览器暂存起来。然后在调用requestAnimationFrame的时候，如果你在一开始做了读取样式属性的操作，那么将会触发浏览器的强制同步布局过程！

有一个方法能同时解决上面的两个问题：对样式修改操作去抖动，控制其仅在下一次requestAnimationFrame中发生：

	function onScroll (evt) {
	
	  // Store the scroll value for laterz.
	  lastScrollY = window.scrollY;
	
	  // Prevent multiple rAF callbacks.
	  if (scheduledAnimationFrame)
	    return;
	
	  scheduledAnimationFrame = true;
	  requestAnimationFrame(readAndUpdatePage);
	}
	
	window.addEventListener('scroll', onScroll);
	

# 黄金外链

谷歌开发文档系列：[https://developers.google.com/web/tools/chrome-devtools/profile/?hl=en](https://developers.google.com/web/tools/chrome-devtools/profile/?hl=en)

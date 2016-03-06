---
layout: post
title: "js中的异步执行时间探究"
description: "js中的异步，js的单线程，浏览器的多进程"
category: tech
tags: [js,异步]
---
{% include JB/setup %}

## 背景

最近调了一份代码，是A文件里发送了一个ajax请求，在回调中触发了aevent事件，即trigger('aevent'),b文件中监听了这个事件,即on('aevent'，function(){}),a文件先于b文件被引用，结果发现在本地调试时，b文件中on语句先调用，a文件中trigger后调用，到了服务器上却成了b文件中on先调用，a文件中trgger语句后调用。

我觉得并不奇怪，因为我的观念里异步请求中的回调函数是在异步请求返回后执行，学长看后怀疑，他说js是异步一定晚于同步语句执行，还给我画了个同步是一条主支，异步是个分支，会被同步主支的末端被合并的图，让我开始深深怀疑异步是否真的一定晚于同步语句执行的问题。

虽说当时那个bug解决了，跟这个问题没关，因为的的语句在requirejs的`require('aModule',function(){})`的回调函数中，而requirejs本身就是一个异步加载，2个异步回调函数谁先执行自然是不确定的。但是我还是想挖一挖同步异步的坑。

## 单线程多进程

都说js是一门单线程多进程的语言，其实是Js是单线程的，一个浏览器进程中只有一个JS的执行线程，同一时刻内只会有一段代码在执行。而异步机制是浏览器的两个或以上常驻线程共同完成的，例如异步请求是由两个常驻线程：JS执行线程和事件触发线程共同完成的，JS的执行线程发起异步请求（这时浏览器会开一条新的HTTP请求线程来执行请求，这时JS的任务已完成，继续执行线程队列中剩下的其他任务），然后在未来的某一时刻事件触发线程监视到之前的发起的HTTP请求已完成，它就会把完成事件插入到JS执行队列的尾部等待JS处理。

JS的主进程调用栈是同步的（单线程，一次只能做一件事），异步调用把回调加入一个队列，等待主栈空闲，Event触发，Event Loop机制把队列里排在前面的回调压入主栈。

## 异步是什么

异步不仅仅包括ajax、setTimeout等。我所认识的异步还有：

1. 回调函数

		function fun1(callback) {
			callback();
		}
		
		function fun2() {
			console.log('我是异步执行函数')
		}
		
		fun1(fun2);

2. 事件机制

	譬如jquery的`tigger/on`,node中的emitEvent模块的`emit/on`。
	
3. 订阅模式

	各种框架实现的sub/pub。
	
4. 异步请求

	原生js的`XMLHttpRequest`。jquery中的ajax。
	
5. promise

6. setTimeout、setInterval。

> "同步模式"就是上一段的模式，后一个任务等待前一个任务结束，然后再执行，程序的执行顺序与任务的排列顺序是一致的、同步的；

> "异步模式"则完全不同，每一个任务有一个或多个回调函数（callback），前一个任务结束后，不是执行后一个任务，而是执行回调函数，后一个任务则是不等前一个任务结束就执行，所以程序的执行顺序与任务的排列顺序是不一致的、异步的。


## 异步的执行时间

对于普通的回调函数、事件机制、订阅模式，异步的执行时间都是确定的。

但是异步请求和setTimout、setTimeInterval的执行时间却是不确定的，原因是他们会让浏览器新开一个进程来完成网络资源请求或者计时任务，完成时回调函数不是立即调用，而是插入到当前任务队列的尾部,等待当前队列中的其他任务执行完毕才执行，至于究竟何时执行是不确定的。

	console.log('start');
	setTimeout(function(){
		console.log('asynchronous')
	},0);
	
	for(var i = 0;i<1000;i++){
		$('.zm-item-answer').append('good');

	}
	console.log('end');
	
见到的打印结果仍然是：

	start
	end
	asynchronous

虽然setTimeout的延迟参数是0，但是他插入不了当前的循环队列中间，只能排到最后，即使循环队列很复杂，耗时长，异步的回调函数也会比它后执行。更不用说耗时长的网络请求了，身边有人测试了网络IO请求消耗时间是内存读取时间的百万倍。网络再好的ajax也快不过同步代码。

## 关于ajax剩下的思考和疑惑

我仍然没办法证明ajax这种异步请求是在请求的网络资源返回后就立即将要执行的回调函数插入到当前任务队列的末端的，也有可能是js有种轮询机制，过了一定时间就会查看一下自己打开的各浏览器进程的执行情况，发现ajax资源返回了再将其回调中的任务插入到任务队列的末端。其实我还是想用某种办法知道究竟是什么情况。

但是我不认同异步代码一定在同步代码执行之后执行的说法，譬如

	function fun1(callback) {
		callback();
	}
	
	function fun2() {
		console.log('我是异步执行函数');
	}
	
	fun1(fun2);
	
	console.log('同步代码');

会打印：

	我是异步执行函数
	同步带吗

只是ajax和setTimeout这种需要浏览器新开进程的异步比较特殊，他们不在当前线程中执行，而是要在某个时间点被插入到js任务队列的末端。总是会被同步代码后执行，特别是ajax请求。这就造成我们会看到这些异步语句总是比所有同步语句后执行。

但是仍有一种说法：js代码中有帧的概念，对于同步代码是在当前帧运行的，异步代码是在下一帧运行的。

我并不清楚js中的当前任务队列里会有哪些内容，也无法知道ajax、setTimeout中回调函数的确切执行时间。但是有一个原则，不能依赖异步执行的时间。

如果同一个js文件中存在2个ajax、setTimeout类的异步语句，这2个异步语句的执行时间总是不确定，完全取决于浏览器先将哪个push到任务队列里，而且每次运行结果都有可能不同。

## 应用实例

看到一个异步运用挺好的例子。

JavaScript是一种单线程执行的脚本语言（这可能是由于历史原因或为了简单而采取的设计）。它的单线程表现在任何一个函数都要从头到尾执行完毕之后，才会执行另一个函数，界面的更新、鼠标事件的处理、计时器（setTimeout、setInterval等）的执行也需要先排队，后串行执行。假如有一段JavaScript从头到尾执行时间比较长，那么在执行期间任何UI更新都会被阻塞，界面事件处理也会停止响应。这种情况下就需要异步编程模式，目的就是把代码的运行打散或者让IO调用（例如AJAX）在后台运行，让界面更新和事件处理能够及时地运行。

	<div id="output"></div>
	
	<button onclick="updateSync ()">Run Sync</button>
	
	<button onclick="updateAsync ()">Run Async</button>
	
	<script>
	
	function updateSync() {
	    for (var i = 0; i < 1000; i++) {
	        document.getElementById('output').innerHTML = i;
	    }
	}
	
	function updateAsync() {
	    var i = 0;
	
	    function updateLater() {
	        document.getElementById('output').innerHTML = (i++);
	        if (i < 1000) {
	            setTimeout(updateLater, 0);
	        }
	    }
	
	    updateLater();
	}
	</script>
	
	
点击"Run Sync"按钮会调用updateSync的同步函数，逻辑非常简单，循环体内每次更新output结点的内容为i。如果在其他多线程模型下的语言，你可能会看到界面上以非常快的速度显示从0到999后停止。但是在JavaScript中，你会感觉按钮按下去的时候卡了一下，然后看到一个最终结果999，而没有中间过程，这就是因为在updateSync函数运行过程中UI更新被阻塞，只有当它结束退出后才会更新UI。如果你让这个函数的运行时间增加一下（例如把上限改为1 000 000），你会看到更明显的停顿，在停顿期间点击另一个按钮是没有任何反应的，只有结束之后才会处理另一个按钮的点击事件。

另一个按钮"Run Async"会调用updateAsync函数，它是一个异步函数，乍一看逻辑比较复杂，函数里先声明了一个局部变量i和嵌套函数updateLater（关于内嵌函数的介绍请看JavaScript世界的一等公民-函数），然后调用了updateLater，在这个函数中先是更新output结点的内容为i，然后通过setTimeout让updateLater函数异步执行。这个函数的运行后，你会看到UI界面上从0到999快速地更新过程，这就是异步执行的结果。

可见，在JavaScript中异步编程甚至是一种必要的编程模式。

## 黄金外链

[js异步之惑](http://blog.whyun.com/posts/js/)

[JavaScript异步编程](https://software.intel.com/zh-cn/articles/asynchronized-javascript-programming)
	
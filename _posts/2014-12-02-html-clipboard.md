---
layout: post
title: "操作剪切板，js 复制链接至剪切板"
description: "关于js flash html5 对剪切板的操作"
category: tech
tags: [tutorial, html5, 基础, js ]
---
{% include JB/setup %}

# 复制链接到剪切板

最近遇到点击复制链接按钮要将一段网址自动复制到剪切板的需求（走了一些弯路，记载一下）。这可是web与系统的交互啊，第一感是纯粹js做不到。上网一搜，果真是，都是js和flash混搭做到这个功能。

不是html5兴起的时代吗，我马上想到去看看h5有类似功能的API不。

## html5
结论：浏览器处于安全的考虑，禁止web层的脚本直接主动操作系统层的剪切板，但是开放了“被动”的API，也就是说，如果用户主动操作了剪切板，发生了copy、paste的事件，你就可以在监听到这个时间后，通过clipboardData对象操作剪切板的内容。

这虽不满足我的需求，但是还是了解下，毕竟按规范办事比较好，不要动不动就用flash。

### 剪贴板事件：

	　　beforecopy：在发生复制操作前触发;
	　　copy：在发生复制操作的时候触发;
	　　beforecut：在发生剪切操作前触发;
	　　cut：在发生剪切操作的时候触发;
	　　beforepaste：在发生粘贴操作前触发;
	　　paste：在发生粘贴操作的时候触发。
	　　


### 不同浏览器下区别
在Firefox、Chrome和Safari中，`beforecopy`、`beforecut`和`beforepaste`事件只会在显示针对文本框的上下文菜单(预期将发生剪贴板事件)的情况下触发。但是IE则会在触发`copy`、`cut`和`paste`事件之前先触发这些事件。至于`copy`、`cut`和`paste`事件，只要是在上下文菜单(右键菜单)中选择了相应选项，或者使用了相应的键盘组合键如(ctrl+v)，所有浏览器都会触发他们。要访问剪贴板中的数据，可以通过clipboardData对象：在IE中，`clipboardData对象`是`window`对象的属性;而在Chrome、Safari和Firefox 4+中，`clipboardData对象`是相应`event`对的属性。但是，在Chrome、Safari和Firefox 4+中，只有在处理剪贴板事件期间，clipboardData对象才有效，这是为了防止对剪贴板的未授权访问;在IE中，则可以随时访问clipboardData对象。为了确保跨浏览器兼容，最好只在发生剪贴板事件期间使用这个对象。

### 函数
`getData(text/url)`:在IE下，text和url参数代表将要从剪切板里获得的数据的保存类型,在Chrome、Safari和Firefox 4+中，实际上是MIME类型

`setData(text/url,data)`:text和url参数代表数据类型,在Chrome、Safari和Firefox 4+中，仍是MIME类型。data是要放到剪切板的内容。

`clearData()`:清空剪切板。

### 测试
W3C标准：[http://www.w3.org/TR/clipboard-apis/](http://www.w3.org/TR/clipboard-apis/)

现在浏览器支持度不是很好。

chrome下这个例子无效[http://jsfiddle.net/c8Ats/7/](http://jsfiddle.net/c8Ats/7/)

	$("#test").on('click', function (e) {
	    var clip = new ClipboardEvent('copy');
	    clip.clipboardData.setData('text/plain', "test");
	    clip.preventDefault();
	
	    e.target.dispatchEvent(clip);
	});
	
但是标准总有一天被执行的么，以后写这个就方便了，不用flash和插件了。


## js+flash
这就只能找插件了。比较靠谱的是[ZeroClipboard](http://zeroclipboard.org/)

github代码库：[https://github.com/zeroclipboard/zeroclipboard](https://github.com/zeroclipboard/zeroclipboard)

之前的版本是要自己引用的ZeroClipboard.swf的，写这篇博客时是2.2.0版本，引用这个swf文件已嵌入到插件内部了，而且支持`jquery`调用。

example:

	<a href="javascript:void(0)" class="copy" data-clipboard-text="http://zeroclipboard.org/">复制链接</a>

将将要复制的内容放到`data-clipboard-text`里。

<script src="../ZeroClipboard/ZeroClipboard.min.js" type="text/javascript"></script>

在元素之后，js调用之前引用插件。

	var client = new ZeroClipboard( document.getElementsByClassName("copy") );

    client.on( "ready", function( readyEvent ) {
         client.on( "aftercopy", function( event ) {
            // `this` === `client`
            // `event.target` === the element that was clicked
            //event.data["text/plain"]:复制的内容
            alert("复制成功：event.data["text/plain"]");
         } );
     } );
     
完结，更多内容请参考插件官网。
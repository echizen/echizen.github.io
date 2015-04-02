---
layout: post
title: "js实现模板替换"
description: "js模板替换，2015阿里春季实习生招聘笔试题"
category: tech
date: 2015-04-01 23:12:43
tags: [pro, js]
---
{% include JB/setup %}


js实现模板替换：

	String.prototype.render = function(arr){
    	var replaceKey = [];
    	var str = [];

    	for(var key in arr){
       		replaceKey.push(key);
        	str.push(arr[key]);
    	}

    	var format = this;

    	for(var i=0; i<replaceKey.length;i++){
        	var reg = new RegExp("\\${" +replaceKey[i]+ "}", "g");
        	format = format.replace(reg,str[i]);
        	console.log(format);
    	}
    	return format;
	}

	var greeting = 'my name is ${name}, age ${age}';
	var result = greeting.render({'name':'jack','age':16});
	console.log(result);
	
	
	
这是阿里2015年实习生招聘的一个笔试题。主要考查了原型继承、正则匹配替换。细节有考查了如何遍历获取json中的见和值，如何在正则中使用变量。当时我是花了20分钟还没写出来，瞎弄了个死版本，觉得这题有意思，于是晚上又思考了下，弄出来了比较灵活的版本。

关于原型继承我是昨天才去看了2篇博客，平时没用过。看到这题后我还是想到是prototype的用法，拓展某个原型，看到题给的是字符串，我在控制台下试出是`String`。

折腾了我比较久的是由于'$'在正则中是结束符，平时写正则是`/\$\{name\}/`就好，但是在`new RegExp()`不行，因为它的第一个参数是字符串，\在字符串中需要转义为\\，所以需要双重转义符`\\`.

在遇到了如何自己用js实现路由，如何自己用js实现数据双向绑定后，我又遇到了如何自己用js实现模板。。。对于我这个长期用框架的人来说，可算是折腾。我也感觉到自己的学习方式有误，不能一贯用框架，应该有自己的思考，才不至于沦陷为搬运工。

**原型闭包什么的高阶js还是要看看，虽然平时不懂也可以写出东西，但是那些是很原始很低级的写法。了解了这些你可以让你的代码变得高质，而且掌握了基础之后，才会发现还可以这么写，方便高效**

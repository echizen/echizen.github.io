---
layout: post
title: "jade学习笔记"
description: "从0开始用jade"
category: tech
tags: [basic, tool]
---
{% include JB/setup %}

很早就想用jade了，最近辞去工作，在家进修，可以好好研究一下想学的东西了。环境最麻烦，所以这次有时间记录一下从0开始使用工具的过程。至于具体写法，官网说的很详细。

# 安装

全局安装：

	sudo npm install jade -g
	
# jade->html

先写一个jade文件，如demo.jade:

	doctype html
	html(lang="en")
	  head
	    title= pageTitle
	    script(type='text/javascript').
	      if (foo) bar(1 + 5)
	  body
	    h1 Jade - node template engine
	    #container.col
	      if youAreUsingJade
	        p You are amazing
	      else
	        p Get on it!
	      p.
	        Jade is a terse and simple templating language with a
	        strong focus on performance and powerful features.
	        
然后在终端运行：`jade demo.jade`

	>jade_test jade demo.jade
	render demo.html
	
会看到在同一个文件夹下生成了`demo.html`，且是压缩版。

如果希望html具有可读性，就不要压缩，运行

	jade -P demo.jade
	
# 文件夹移动

如果是实际用的话，肯定要考虑分文件夹。

如我们在jade_test下新建文件夹jade和html。jade文件夹下新建test.jade文件。运行

	jade jade/test.jade --out html
	
就会在html文件夹下发现被渲染的test.html。

也可以进入jade文件夹下运行

	jade test.jade --out ../html
	
不想压缩就运行：

	jade test.jade -P --out ../html
	
# 监听文件变化

运行以下命令，可以让test.jade变化后,test.html随时更新

	jade test.jade -P -w --out ../html
	
可以同时监听多个文件变化，在test_jade文件夹下

	jade jade -P -w --out html
	
但是新增文件时要重启才有效果，监听不了文件的增删
	
# subilime工具

一开始jade文件在sublime里是惨白色，没有语法高亮肯定不合习惯。我们可以装点jade插件.

![image](https://echizen.github.io/assets/blog-img/QQ20150811-1@2x.png)

发现bootstrap真强大，都有jade的snippet了。

我装了第一个jade,现在有语法高亮了。

# 使用细节

## 换行

 div内内容过长时会需要换行，换行是`.+space`,eg:
 
	 - var divClass=['fluied','text-left']
	      div(class=divClass).
	        重名的class会发生什么
	        后者覆盖前者。
	        换行是通过'.'+' '实现
	        
## 空格的重要性

      ul#explict
        -
          list = ["Uno", "Dos", "Tres","Cuatro", "Cinco", "Seis"]
        each item in list
          li= item
                    
  这种东西,`-`和`list`之间只有一个换行符，不能有空格。`each`必须顶格，`li`和`=`之间必须没有空格。
  
## markdown filter

	Transformers.markdown is deprecated, you must replace the :markdown jade filter, with :markdown-it and install jstransformer-markdown-it before you update to jade@2.0.0.
	
## include 和 extends同时使用

公用头在非单页面应用中是非常中要的，extends可以让我们继承模板，include可以引入外部模板，于是我们便可以这么干：

公用基础

	//- comm.jade
	doctype html
	html
	  head
	    block title
	      title 公用头
	
	    block script
	      include lib.jade
	  body
	    block content

资源：

	//- lib.jade
	link(href='/lib/bootstrap/css/bootstrap.min.css',rel='stylesheet')
	link(href='/home/css/style.css',rel='stylesheet')
	script(src='lib/angular.js')    

具体页面

	//- index.jade
	extends  comm.jade
	
	block title
	  title 首页
	
	block content
	  h3 欢迎光临
	
	  div.fluid.
	     jade用起来感觉不错
	     但是貌似更适合node渲染的页面，而不是mvc的页面
	  script
    	include ../js/index.js
    	
资源：

	// index.js
	alert('jade is awesome');

**注意：**comm.jade中必须用`block script`这种方式，将要引入的文件定义为block,才能在index.jade中正确引入。

最后那个资源引入真的很爽，你可以在把js抽出一个新文件，但是渲染时依然被拉入html中作为内嵌js了。我们很多时候抽出js都只是为了开发人员方便，对计算机最友好的方式其实是都写在一个页面，这样可以避免需要再一次通过网络请求资源。

# 感受

敲了半天的感受就是jade让前端像写js一样写html，能定义变量，使用变量。但是应用场景貌似更适合node模板引擎先渲染，而前后端完全分离的前端mvc模式并不显其优势。作为被node先渲染的模板，可以将一些变量先传入，再有jade语法做功能判断，可是实现动态html。而前后端完全分离后，html中不会插入任何后台代码，这样变量的作用就小了一半。

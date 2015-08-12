---
layout: post
title: "less 学习笔记"
description: "从0学习less，less的环境"
category: tech
tags: [basic,tool]
---
{% include JB/setup %}

乘热打铁，今天把less也看了。

##安装

	npm install -g less
	
##运行

	lessc less/style.less > css/style.css
	
这里必须指定生成的css路径，否则会在终端输出结果。。。

压缩的话

	lessc -x less/style.less > css/style.css
	
更科学的是装clean-css插件。

你也可以选择不将less转化成css，把这个工作交给客户端浏览器。（个人觉得增加了浏览器负担，使渲染速度变慢）

	<link rel="stylesheet/less" type="text/css" href="styles.less" />
	<script src="less.js" type="text/javascript"></script>
	
##安装sublime less插件

shift+cmd+p -> install package -> less

装完要重新打开文件或重启sublime才能生效

这是为了语法高亮，要是想保存时能自动把less转化成css，也有插件可装

## less-plugin-clean-css

css不压缩还是不能接受的。

-x的用法是被遗弃的，所以就要装插件。

	sudo npm install -g less-plugin-clean-css
	
运行：

	lessc less/style.less > css/style.css --clean-css
	
##实时编译

less的实时编译需要借助其他工具，如gulp的watch模块，koala...
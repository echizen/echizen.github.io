---
layout: post
title: "webpack-dev-server使用react-router browserHistory的配置"
description: ""
category: tech
tags: [react webpack tools]
---
{% include JB/setup %}

本来这玩意不值得写一篇博客的，但是坑爹的是我在网上搜了一圈解决方案都不可行，webpack-dev-sever官网的文档使用说明还有误导性。最后还是自己慢慢拨出来的。

## 问题的产生

###  browserHistory如何设置路由

react-router有3种路由方式：贴近实际路由的browserHistory，我们熟悉的兼容性很好的hashHistory，还有我还没了解的createMemoryHistory。

browserHistory使用的是h5 history api，给用户看到的浏览器url与普通的path没有区别，其实路由都是交给js来处理的，后端只用把所有根路径下子path的路由都指向同一个文件，交由js根据path处理展示内容，js通过history.push、history.repalce等api处理链接跳转，其实用户看到的只是url被repalce，页面并没有刷新，js做了动态替换了dom，获取数据等等，但是视觉上感觉页面跳转了，速度更快，除了数据展示部分页面没啥闪烁，当然你也可以做一些渐入渐出的特效让“页面跳转”看起来更自然，扯远了。。。


### webpack-dev-sever如何定位文件

browserHistory的路由跟普通url path部分没啥2样，通过'/'分割路径。

webpack-dev-sever是静态资源服务器，他会通过你的output配置去读取文件，通过'/'分割以文件查找的模式匹配文件。这样自然就产生问题了，因为你配置的路由并不是实际存在的文件，根据文件查找的方式是找不到的，只会404。

## 解决方案

### webpack.config.js配置

webpack有一个选项devServer用于协助wepack-dev-server支持h5 history api的路由。于是你可以类似这样的配置：

	module.exports = {
	    entry: "./src/app/index.js",
	    output: {
	        path: path.resolve(__dirname, 'build'),
	        publicPath: 'build',
	        filename: 'bundle-main.js'
	    },
	    devServer: {
	        historyApiFallback:{
	            index:'build/index.html'
	        },
	    },
	    //其他的配置省略
	};

output.publicPath告诉webpack从该路径下寻找文件，在这里也就是把build当成文件系统入口，根路径。

devServer.historyApiFallback的意思是当路径匹配的文件不存在时不出现404,而是取配置的选项historyApiFallback.index对应的文件。

这样开启webpack-dev-server后会看到终端：

![image](https://echizen.github.io/assets/blog-img/20160705.png)

ps:我就是修改配置项看终端信息才知道这2个参数是这个意思，坑爹的官网解释就是个坑，遇到问题已经不能只靠官网和stackoverflow了，还是多自己研究原理和现象吧。

### 坑爹的官网

[http://webpack.github.io/docs/webpack-dev-server.html#the-historyapifallback-option](http://webpack.github.io/docs/webpack-dev-server.html#the-historyapifallback-option)

![image](https://echizen.github.io/assets/blog-img/QQ20160705-0@2x.png)

当然这也有可能不是官网文档的信息有问题，有可能是我webpack-dev-server是1.10.1，而官网最新版是1.13的问题。

我按官网这种将historyApiFallback.index和output.publicPath配置为同一个路径，webpack-dev-server是不知道去找这个文件夹下的index.html或者其他html文件的。


还有一个地方看官网文档纳闷了一下，顺便解释。

- module.exports透出配置是使用终端CLI启动（或者类似npm script配置利用npm run启动这类命令行启动）webpack-dev-server。

	module.exports = {
	    // configuration
	};
	
- webpack({})这种方式是给node.js api使用的。


		var WebpackDevServer = require("webpack-dev-server");
		var webpack = require("webpack");
		
		var compiler = webpack({
		  // configuration
		});
		var server = new WebpackDevServer(compiler, {
		  // webpack-dev-server options
		
		});
		
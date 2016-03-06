---
layout: post
title: "浅谈hybrid APP前端架构"
description: "浅谈hybrid APP前端架构，使用Framework7构建APP前端框架，MVC架构"
category: tech
tags: [ pro, 前端]
---
{% include JB/setup %}

随着这个hybrid 项目接近发版，我想写点什么。这个项目最大的收获是亲身实验了mvc的架构吧，其他的都是搬运工，搬运F7封装好的一些功能效果。不过架构还是有可取之处的。

先不说hybrid和mvc的好处了，网上一片一片的。我只说最显著的，hybrid app很灵活，网页部分你可以天天更新，每天优化，免去了发版升级。mvc架构更清晰，与后台交互成本变低，缓存也变得可能，网页性能上升。

## 感受


1. 单页面：像Framework7、Iconic 都是单页面应用框架，但是做hybrid APP时，不可能是单页面，与原生页面的切换结合注定我们不可能是单一入口的单页面，这些页面有多个入口，但对于每个入口进入的一系列web页面，都是单页面。

2. 缓存：手机端加载页面很慢，架构时一定要考虑到缓存，js的通用资源要放一起。能确定不变的东西要剥出来。

3. requiredJS：影响页面速度的主要是js的加载和渲染，所以js一定不要手写，一便又一遍的请求。单页面时，所有“页面”都留在dom中，返回事件或者js端的逻辑跳转页面会让我们一遍又一遍的访问同一个页面，或者前后页面有公用的js，用requiredJS可以保证资源只被加载一次，不会重复请求加载。

4. 刷新：缓存的矛盾体。我的原则是用户的刷新行为永远只刷新数据，所以在controller里我会剥离出dataInit独立的请求数据资源，刷新时只调用他。


## modal

这是nodeJs在做，nodeJS可以跟服务器沟通，又与前端JS无缝连接，是一座很好的桥梁，用它来封装数据、处理逻辑是极好的，很遗憾这块不是我写的。

## view

结合F7的框架结构，我抽出来了公用的头和尾，其它的页面仍然是一张一张的，其实单也页面应用并不是真的全都写在一个文件里，只是在浏览器下页面间不是通过url跳转，而是通过ajax请求后放在同一页面内。

	<!DOCTYPE html>
	<html class="<%=ua.Android != undefined?'android android-4':''%>">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no, minimal-ui">
        <meta name="apple-mobile-web-app-capable" content="yes">
        <meta name="apple-mobile-web-app-status-bar-style" content="black">
        <meta name="format-detection" content="telephone=no" >
        <title>XXX</title>

        <!-- Path to Framework7 Library CSS-->
        <link rel="stylesheet" href="f7/css/framework7.min.css">
        <!-- Path to your custom app styles-->
        <link rel="stylesheet" href="f7/css/my-app.css">
    </head>
    <body>
        <!-- Status bar overlay for fullscreen mode-->
        <div class="statusbar-overlay"></div>
        <!-- Views-->
        <div class="views">
            <!-- Your main view, should have "view-main" class-->
            <div class="view view-main">
            
   头文件的特色就是不加JS！！！，对，js严重影响加载和渲染速度，在手机端很明显。css我都写到了一个文件里，不过也不是很大，这个我自己也不知道这是不是个好的行为。
   
           		</div>  
    		</div>
  		</body>
	</html>

	<!-- Path to Framework7 Library JS-->
	<script type="text/javascript" src="f7/lib/framework7.min.js"></script>
	<!-- js-sdk 文件路径 -->
	<script type="text/javascript" src="f7/lib/jhyx-1.0.0.js"></script>
	<!-- 本地存储文件 -->
	<script type="text/javascript" src="f7/lib/store.min.js"></script>
	<!-- Path to your require js-->
	<script data-main="f7/app" src="f7/require.js"></script>
	
尾的特色是只加载框架js，通用js和requiredJS，其他的零部件都交给requiredJS去按需加载吧。

每个页面为了能解析必要的nodeJS(毕竟咱不是webapp)，都是ejs文件，但是用户在浏览器上看到的url是html,在node端的app.js替换掉html,变成静态页面路由。并且对设备类型进行了检测。


	/**
	 * Js-Bridge , 静态页面路由
	 */
	app.get('/*.html',function(req,res){
    var curUrl = req.protocol + '://' +req.host + req.url;
    var query = req.query;

    if (query.layout == undefined || query.layout == 'false') {
        layout = false;
    }else{
        layout = true;
    }

    // 浏览器头检测
    var ua = req.headers['user-agent'],$ = {};

    if (/mobile/i.test(ua))
        $.Mobile = true;

    if (/like Mac OS X/.test(ua)) {
        $.iOS = /CPU( iPhone)? OS ([0-9\._]+) like Mac OS X/.exec(ua)[2].replace(/_/g, '.');
        $.iPhone = /iPhone/.test(ua);
        $.iPad = /iPad/.test(ua);
    }



    if (/Android/.test(ua))
        $.Android = /Android ([0-9\.]+)[\);]/.exec(ua)[1];

    if (/webOS\//.test(ua))
        $.webOS = /webOS\/([0-9\.]+)[\);]/.exec(ua)[1];

    if (/(Intel|PPC) Mac OS X/.test(ua))
        $.Mac = /(Intel|PPC) Mac OS X ?([0-9\._]*)[\)\;]/.exec(ua)[2].replace(/_/g, '.') || true;

    if (/Windows NT/.test(ua))
        $.Windows = /Windows NT ([0-9\._]+)[\);]/.exec(ua)[1];


    
    if (/MicroMessenger/.test(ua) && query.weixin == "true") {
        var mou_sign = require('cloud/mou_sign');
        weixin = true;

        mou_sign.signJson(curUrl).then(function(wxconfig){
            wxconfig = JSON.stringify(wxconfig);
            res.render('f7' + req.path.replace('.html',''),{layout:layout,ua:$,weixin:weixin,wxconfig:wxconfig});
        });

    }else{

        weixin = false;
        wxconfig = JSON.stringify({});
        res.render('f7' + req.path.replace('.html',''),{layout:layout,ua:$,weixin:weixin,wxconfig:wxconfig});
    }
    
	});
	
**其实像Framework7、Iconic 都是单页面应用框架，但是做hybrid APP时，不可能是单页面，与原生页面的切换结合注定我们不可能是单一入口的单页面，这些页面有多个入口，但对于每个入口进入的一系列web页面，都是单页面。**

这样的逻辑，加上微信端对页面的应用，让我们的一些页面可能是入口页面也有可能是非入口页面，入口页面是要加载头和尾的，非入口页面却不能，所以我们在app.js设置了layout 变量，通过url赋值。然后在每个页面做判断。

	<% if(layout)/{/%>
	<% include comm/header %>
	<% /}/%>
	......
	<% if(layout)/{/%>
	<% include comm/footer %>
	<% /}/%>

(由于与jekyll模板冲突，所以我将大括号+百分号转义了)

## controller

最最精华的controller部分来了，它处理页面动效、用户交互、模板渲染、路由导航。

我是给每个页面都有独立的js模块的，然后用requiredJS调用公用模块的js.每个页面的js加载规则在route.js中配置。

js文件的结构部分截图：
![image](https://echizen.github.io/assets/blog-img/QQ20150329-2.png)

虽然有点繁，我给每个页面都建了个文件夹，里面就一个controller文件，其实这是考虑，可能我们后期会把modal部分也从nodeJS端迁到js来做。由于约定了文件夹和js文件与页面文件同名，所以规则就很好配置了。

	/**
     * Load (or reload) controller from js code (another controller) - call it's init function
     * @param controllerName
     * @param query
     */
    function load(controllerName, query) {
        if (ignoreList.indexOf(controllerName) == -1) {
            require(['js/' + controllerName + '/'+ controllerName + 'Controller'], function(controller) {
                controller.init(query);
            });
        };
    }


先看看静态资源的构成。

![image](https://echizen.github.io/assets/blog-img/QQ20150329-1.png)

-require.js:不用说，requireJS

-app.js:从尾部文件的代码里也可看到，这是js入口，它有公用模块的config信息。有载入router.js模块并调用他的入口，有初始化f7框架、模板编译函数、mainView初始化.

-JS文件夹
	-router.js:监听f7的pageBeforeInit()状态，完成对url参数的解析判定，调用load函数，加载相应的js controller,并执行controller。
	-pagename文件夹
		-pagenameController.js:依据页面需求完成数据请求、页面模板解析、事件绑定。
		
-lib文件夹：装一些库和插件，如jquery,验证插件什么的，是公用通用静态资源。lib文件夹将在app中做本地缓存。

## 关于缓存

为了速度，静态资源都做本地缓存，譬如css文件、img文件、lib文件夹下的资源，以及一些没有任何数据的静态页面。

## 关于路由

其实这个是F7做好的，具体的不清楚，但是可以肯定的是js路由就是：

1.监听用户行为,如点击链接，或者js端逻辑调用

2.发送ajax请求获取一个一个的页面内容按照一定的结构规则添置到入口页面。这些页面依旧有自己的url，但是你在浏览器上看到url确实不变的，其实他是hash导航，通过hash参数判断自己当前是那个文件。
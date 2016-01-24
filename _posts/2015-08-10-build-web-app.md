---
layout: post
title: "web app开发心得"
description: "用angularjs搭建web app，ionic框架实战"
category: tech
tags: [js, angularjs, pro]
---
{% include JB/setup %}

#背景
在朝思暮想之后，终于在7月份有机会将angularjs投入项目了！！！本来头是不乐意的，说我们之前的hybrid app已经拥有一套成型的技术了，为什么还要折腾。可我想说折腾是程序员的本性，如果不试新东西怎么会进步呢？一遍又一遍的跑同样一套系统对于我来说是无聊。。。于是我便忽悠老大，angular和ionic都是谷歌开发出来的，技术上多么多么的先进。之前的那套架构有多么多么多的漏洞。于是老大心不甘情不愿的让我去试了，我也做了担保，要日夜兼程的按期完成。

说起这一次的项目，折腾的真是够多的。项目不算大，但是涉及了UI完全不同的三个平台。先是app端，架构师用phonegap封装，内部页面我也就用angular和ionic这2大红的发紫的框架了。pc端管理后台，因为想看看angular和jquery合作带来的问题怎么解决，然后考虑到另一个做这个前端css技能较为薄弱，所以采用了adminLTE+angularjs。用户展示官网，采用了angularjs+bootstrap,也就是bootstrap-ui,这也是一个相当火的bt新产品，然后由于我想写写ui了，就用了bt，毕竟bt的自定义能力很强。

###项目中的我
这次非常开心，拉了2个协会的小学弟，虽然他们2没参与过正式项目，整体化意识有欠缺，但是协会的人的学习能力是毋庸置疑的。还有一个小学弟，是刚从php转前端的，pc后台交给他去完成，虽然前期慢了点，bug幼稚了点，但还是像模像样的完成了，吾心甚慰。

至于我的角色。。。技术顾问+打杂的。。。我先把架构搭好，然后每个平台，抽一个典型页面，写个demo，前后端调通，再交给他们参考写其他。他们解决不了的技术问题，都扔给我，或者调bug，进度赶时或者页面复杂时，我就会操刀写页面。曾经一度十几天吧，我都处于各种解决技术问题中，因为自己也是第一次用这些，没经验，很多问题也没头绪，就去看angular官网，了解原理，按自己的理解试一下，真调不出来就去stackoverflow上找别人的参考。天天深陷Bug和难题中比较绝望，但是解决很有质量的问题很有意义。在每天解决难题的煎熬和没什么新的问题之后，我就去写官网了。阔别多月之后再次感叹bt真的是世界上最好的前端框架，既有规范，又很自由，这套框架没有多写一句没用的代码，你完全可以在他的基础上构建自己的UI样式，它提供的只有便捷。

#脚手架

第一次自己架构一个项目，架构完后才知道为什么有yoman这种一体化的脚手架。

这起源一个问题：

##angularjs如何按需加载

之前的一个项目是用requirejs按需加载的，深感按需加载很必要。

但是用angularjs就会发现，无论是用<ng-include>还是ui-router,所有html和js在你打开第一个页面时就一股脑加载进来了。。。这是多么的不可接受。

我也试过用requirejs和angulrAMD/oclazyload这些东西尝试动态加载，但是不是只能动态加载html就是只能动态加载js，要不就是没法和ionic整合。

还去知乎提了问题。其中有人回答我不需要动态加载，压缩后一次性加载也没问题。然后我开始向着这个方向进军。但是在还不知道gulp、grunt工具的我很难想象所有人在同一个js中写文件，冲突什么的有多可怕。。。

##文件和网络请求数，谁更耗时

这个问题要认真想，对于我们的项目，每个页面都不是很复杂，js代码500行之内绝对能搞定，但是页面多，如果你给每个页面都写一个contrllor.js的话，可知跑完一个app，用户得请求多少次网络。而我们将所有js文件分类打包成一个文件并压缩，这样不同类的js文件加起来才不到50k，而服务器再使用gzip压缩这些文件，就更小了，根本不恐怖。那么问题是如何让开发过程顺利，总不能所有人真的往一个文件里写东西吧。。。

##工具时代

###gulp

然后我找到了grunt,不久因为灵活性差，又换成了gulp。gulp能在这个项目里起很重要的作用：

1. 从bower下载的资源文件抽出开发要用的文件，打包到开发目录。
2. 将多个js合并成一个js
3. 将js文件压缩

而gulp的用法是简洁明了。非常方便。

如资源文件移动：

	/**
	 * 前端资源文件打包
	 */
	var BOWER_COMPONENTS_PATH = {
	  'src':'./bower_components/',
	  'dest':'./static/lib/'
	};
	gulp.task('bower-ionic',function(){
	  var tempPath = BOWER_COMPONENTS_PATH.src + 'ionic/release/';
	  return gulp.src([tempPath + '**/+(*.min.*|*.eot|*.svg|*.ttf|*.woff)'])
	    .pipe(gulp.dest(BOWER_COMPONENTS_PATH.dest + 'ionic/'));
	});

再如文件合并压缩

	/**
	 * 控制器文件编译打包
	 */
	var ANGULAR_BUILD_PATH = {
	  src:'./build/',
	  dest:'./static/'
	};
	gulp.task('pa-angular-ctrl-app',function(){
	  return gulp.src([ANGULAR_BUILD_PATH.src + 'app/controllers/**/*_ctrl.js'])
	    .pipe(plugins.jshint())
	    .pipe(plugins.jshint.reporter('jshint-stylish'))
	    .pipe(plugins.uglify())
	    .pipe(plugins.concat('app_ctrl.js'))
	    .pipe(gulp.dest(ANGULAR_BUILD_PATH.dest + 'app/js/'));
	});

还可以将多个任务合成一个任务，因为你不可能一次只合并一类文件：

	gulp.task('pa-angular',['pa-angular-ctrl-app','pa-angular-sev-app','pa-angular-ctrl-pc','pa-angular-resource-services','pa-angular-directives-app','pa-angular-directives-pc','pa-angular-directives-home','pa-angular-filters-app','pa-angular-filters-pc','pa-angular-ctrl-home']);

###bower

现在UI越来越重要，那么多界面效果，所需的插件肯定不少。而bower就是必需品。没有bower，你能想象更新插件是多么麻烦吗？

1. 下载资源文件
2. 更新资源文件
3. 删除资源文件

将bower和gulp一起运用，可以在插件有更新时使用'bower update XXX'一键更新，然后运行gulp命令，将新的插件资源移动到开发目录。

此时若更新后，插件目录有变化，gulp命令就要重写，不能忽略了检验。

#架构
现在谈谈核心内容了。

angular架构app分成5个部分：页面html，控制器controller，服务service，指令directive，过滤器filter.

##路由

ionic使用的是ui-router,我便把pc官网也引入了ui-router。由于单页面，只有一个ng-app注入模块，所以我们可以将其他模块作为ng-app注册模块的依赖。如：

	"user strict";
	 // Declare app level module which depends on views, and components
	 var jbj = angular.module('jbj', [
	    'ui.router',
	    'jbj.news',
	    'jbj.newsDetail'
	   ]);
	
	jbj.config(['$stateProvider', '$urlRouterProvider',function($stateProvider, $urlRouterProvider) {
	
	  //angular路由配置
	  $stateProvider

	  .state('news', {
	    url: '/news',
	    templateUrl: 'news.html',
	    controller: 'newsCtrl as news'
	  })
	
	  .state('newsDetail', {
	    url: '/newsDetail/:objectId',
	    templateUrl: 'news_detail.html',
	    controller: 'newsDetailCtrl as newsDetail'
	  })
		
	  $urlRouterProvider.otherwise('/index');
	
	}]);
	
然后在页面上只用注入 ` <body ng-app="jbj" ng-cloak>`即可。

##页面
这个没什么可介绍的，就是分平台建文件夹，一个页面一个文件的传统做法。

##controller
也就是普通的contoller，相关的部分放在一个模块中，其他的是每个页面自己的特色化js部分，调用directive、service、filter时，记得引入模块，需要注入依赖的就注入依赖。

##service
由于接口要统一，三个平台使用了angularjs，都是数据双向绑定模式，所以，seviece采用统一的。service有2类，一类是数据接口，一类是工具。

数据接口的service没有使用$http服务，而是使用了更适合RESTful风格的$resource。

你看到的将会是这样，一个url指定所有增查改删的操作。

	  var apiUrls = {
	    news: WEB_HOST_URL + '/api/1.0/News/:id',
	    imgUploadUrl:  WEB_HOST_URL + '/api/1.0/File/uploadBase64Img',
	  };
	
	  var newsService = angular.module('newsService',['ngResource']);
	
	  newsService.factory('News',['$resource',function($resource){
	    return $resource(apiUrls.news,{id:0},{
	        get:{method:"GET"},
	        post:{method:"POST"},
	        delete:{method:"DELETE"},
	        put:{method:"PUT"}
	    });
	  }]);
	  
工具类的接口我放的都是为了能复用的但又跟dom无关的功能性代码，如phonegap和微信的调取图片的接口。

##directive、filter

通用的跟ui和js都有关系的部分，我是将所有directive都放在同一个文件夹内，通过后缀区分。譬如只给app用的叫做XXX_directive.app.js。给pc后台用的叫XXX_directive.pc.js，通用的叫XXX_directive.js。然后根据后缀名合并文件。

	gulp.task('pa-angular-directives-app',function(){
	  return gulp.src([ANGULAR_BUILD_PATH.src + 'directives/*_directive.js',ANGULAR_BUILD_PATH.src + 'directives/*_directive.app.js'])
	    .pipe(plugins.jshint())
	    .pipe(plugins.jshint.reporter('jshint-stylish'))
	    .pipe(plugins.uglify())
	    .pipe(plugins.concat('jbj_app_directives.js'))
	    .pipe(gulp.dest(ANGULAR_BUILD_PATH.dest + 'directives/'));
	});
	
	gulp.task('pa-angular-directives-pc',function(){
	  return gulp.src([ANGULAR_BUILD_PATH.src + 'directives/*_directive.js',ANGULAR_BUILD_PATH.src + 'directives/*_directive.pc.js'])
	    .pipe(plugins.jshint())
	    .pipe(plugins.jshint.reporter('jshint-stylish'))
	    .pipe(plugins.uglify())
	    .pipe(plugins.concat('jbj_pc_directives.js'))
	    .pipe(gulp.dest(ANGULAR_BUILD_PATH.dest + 'directives/'));
	});

filter的处理方式相同。

##发行版

要把开发目录的js内容打包到发行目录下，譬如所有app端controller合成一个app_ctrl.js文件，在index.html引入的也是发行目录下打包后的app_ctrl.js。

为了方便开发人员实时看到效果，还要实时更新合并文件。gulp中有watch服务，负责监听文件变化，注入一下就好：

	gulp.task('pa-angular',['pa-angular-ctrl-app','pa-angular-sev-app','pa-angular-ctrl-pc','pa-angular-resource-services','pa-angular-directives-app','pa-angular-directives-pc','pa-angular-directives-home','pa-angular-filters-app','pa-angular-filters-pc','pa-angular-ctrl-home']);
	gulp.task('watch-pa',function(){
	  plugins.livereload.listen();
	  gulp.watch([ANGULAR_BUILD_PATH.src + '**'],['pa-angular']);
	});
	
运行`gulp pa-angular`后，被监听的目录发生变化，pa-angular的任务就会被触发，一群纳入控制的文件就讲被合并打包，我建议为了开发时调试，开发时打包不要压缩。

#给力的node端-本地服务器
我不玩node的，你的node小伙伴需要给力的给你一个本地跑的起来的服务器，你才能方便的开发。如果node小伙伴不给力，就自己折腾一个服务吧，现在也有不少这种的，真不行，通过`python -m SimpleHTTPServer 8888`通过文件访问也可以啊

#我的项目结构

	--bower_components
	--build
		--app
			--**_ctrl.js:app端各个页面的controller文件
		--pc
			--**_ctrl.js:pc后台各个页面的controller文件
		--home
			--**_ctrl.js:官网各个页面的controller文件
		--directives
			--**_directive.js
			--**_directive.app.js
			--**_directive.pc.js
			--**_directive.home.js
		--filters
			--**_filters.js
			--**_filters.app.js
			--**_filters.pc.js
			--**_filters.home.js
		--services
			--api：接口服务
				--**_service.js
			--util：工具服务
				--**_service.js
	--static
		--app
			--css
			--img
			--icon
			--js
				--router.js:路由和模块初始化
				--app_ctrl.js:gulp合并而来
		--pc
			--同上
		--home
			--同上
		--directives:打包而来
			--app_directives.js
			--pc_directives.js
			--home_directives.js
		--filters:打包而来
			--app_filters.js
			--pc_filters.js
			--home_filters.js
		--services:打包而来
			--service.js
		--lib
			--**:各种经gulp分包后的插件
	--views
		--app
			--**.html
		--pc
			--**.html
		--home
			--**.html
	--bower.json
	--gulpfile.js
	--package.json
	--.gitignore
	

#待优化

如果你时间充裕的话，应该试试karma这种测试工具，写测试，毕竟angular的原则是测试先行。只是我还没有领悟其中奥妙，service、directive、controller都可以写测试，时间跟不上。

css方面试试less,可以节省不少时间

		
---
layout: post
title: "webpack-源码分析"
description: "webpack-源码分析"
category: tech
tags: ['webpack']
---
{% include JB/setup %}

# 感受

先说下阅读感受，一开始打开webpack仓库，看到100多个文件，我的内心是懵逼的。然后看到一大半都是plugin，心中窃喜这些应该是不需要了解的。再后来读着读着发现很多想了解的功能实现都是通过plugin的。。。

webpack的代码真的还是很复杂的，感受最深的是其利用tapable来实现的一套插件体系，扩展性很好，设计者还是很机智的，在这套体系上，内部近百个插件有条不紊，还能支持外部开发自定义插件来扩展功能，如果不是这个机制，整个庞大的构建流会更迷吧。支持这么多功能的构建工具开发出来复杂度真的挺高的。

主流程还是比较清晰的在compiler里，但是去看compiler和compilation的生命周期就各有几十个，每个调用一批插件协调处理，如果不去debug，这个事件触发机制根本就找不到会触发哪些插件的事件回调，根本无法也没有意义一一看完，只能说后面有需要时再看具体的了。

抓住主线搞清整体流程，`make`, `seal`, `render`，`emitAssets`这些核心步骤实现，可以通过通过几个常用的loader和plugin了解了下这两部分的实现。

我向往的境界是读了能了解设计思路，放开徒手也能写出核心逻辑代码实现，再以后对新东西能自行通过api特征就推断出实现方式，然后读源码验证一下。显然差的还远，目前也就勉强够用，能了解到实现方式，能够知道去哪一块debug来解决遇到的问题，能够写loader和plugin扩展个性化功能。我没有在一堆源码里迷失主线，活着出来了已是万幸了。。。

# 概况

初始化配置参数 -> 绑定事件钩子回调 -> 确定Entry逐一遍历 -> 使用loader编译文件 -> 输出文件

网上有个复杂的图，画清楚了这个步骤：


![](https://s10.mogucdn.com/mlcdn/c45406/190317_2b4k7ad7k75468g03b1eagca24j19_4436x4244.png)

webpack主要的2个部分是Compiler和Compilation，Compiler基本上只是执行最低限度的功能，以维持生命周期运行的功能。它将所有的加载、打包和写入工作，都委托到注册过的插件上。它只是构建任务调度器，而compilation则是具体的构建内容步骤

# tapable机制

要搞清webpack，还是先老老实实花时间搞清楚tapable机制，否则没法理解插件是怎么注册触发的。

tapable提供类似的插件接口。webpack 中许多对象扩展自 Tapable 类。这个类暴露 tap, tapAsync 和 tapPromise 方法，可以使用这些方法，注入自定义的构建步骤，这些步骤将在整个编译过程中不同时机触发。

tapable提供了很多类型的钩子，如同步的`SyncHook`和异步的`AsyncHook`。我们一开始创建钩子类型和可获取参数，相当于规范了这个钩子的模样，然后在这个钩子上注册插件，后续再在触发这个钩子上的插件，传入规范定的参数，进而触发插件注册的处理函数。

example:

```javascript
// 创建钩子
class Car {
	constructor() {
		this.hooks = {
			accelerate: new SyncHook(["newSpeed"]),
			brake: new SyncHook(),
			calculateRoutes: new AsyncParallelHook(["source", "target", "routesList"])
		};
	}
  // 调用对应的钩子，会触发注册的插件回调函数
  setSpeed(newSpeed) {
		this.hooks.accelerate.call(newSpeed);
	}
  useNavigationSystemPromise(source, target) {
		const routesList = new List();
		return this.hooks.calculateRoutes.promise(source, target, routesList).then(() => {
			return routesList.getRoutes();
		});
	}

	useNavigationSystemAsync(source, target, callback) {
		const routesList = new List();
		this.hooks.calculateRoutes.callAsync(source, target, routesList, err => {
			if(err) return callback(err);
			callback(null, routesList.getRoutes());
		});
	}
	/* ... */
}

// 注册插件
const myCar = new Car();
// Use the tap method to add a consument
myCar.hooks.brake.tap("WarningLampPlugin", () => warningLamp.on());
myCar.hooks.accelerate.tap("LoggerPlugin", newSpeed => console.log(`Accelerating to ${newSpeed}`));
myCar.hooks.calculateRoutes.tapPromise("GoogleMapsPlugin", (source, target, routesList) => {
	// return a promise
	return google.maps.findRoute(source, target).then(route => {
		routesList.add(route);
	});
});
myCar.hooks.calculateRoutes.tapAsync("BingMapsPlugin", (source, target, routesList, callback) => {
	bing.findRoute(source, target, (err, route) => {
		if(err) return callback(err);
		routesList.add(route);
		// call the callback
		callback();
	});
});

```

详情可见[https://webpack.docschina.org/api/plugins/#tapable](https://webpack.docschina.org/api/plugins/#tapable)

代码[https://github.com/webpack/tapable](https://github.com/webpack/tapable)

## webpack中的hooks

利用tapable这样的设计就做到了将webpack功能碎片化，每个阶段拆成一个hooks，然后各种内外部插件都可以往这个hooks上注册插件，webpack在编译过程中的合适阶段会调用对应的钩子，从而触发一系列挂载在这个钩子上的插件。这样让webpack的每个构建过程的功能都可以无限扩展和定制。实现了一个强大的系统。

如complilation上注册了几十个hooks，在编译的不同阶段会被触发:

```javascript
class Compilation extends Tapable {
	/**
	 * Creates an instance of Compilation.
	 * @param {Compiler} compiler the compiler which created the compilation
	 */
	constructor(compiler) {
		super();
		this.hooks = {
			/** @type {SyncHook<Module>} */
			buildModule: new SyncHook(["module"]),
			/** @type {SyncHook<Module>} */
			rebuildModule: new SyncHook(["module"]),
			/** @type {SyncHook<Module, Error>} */
			failedModule: new SyncHook(["module", "error"]),
			/** @type {SyncHook<Module>} */
			succeedModule: new SyncHook(["module"]),

			/** @type {SyncHook<Dependency, string>} */
			addEntry: new SyncHook(["entry", "name"]),
			/** @type {SyncHook<Dependency, string, Error>} */
			failedEntry: new SyncHook(["entry", "name", "error"]),
			/** @type {SyncHook<Dependency, string, Module>} */
			succeedEntry: new SyncHook(["entry", "name", "module"]),

			/** @type {SyncWaterfallHook<DependencyReference, Dependency, Module>} */
			dependencyReference: new SyncWaterfallHook([
				"dependencyReference",
				"dependency",
				"module"
			]),

      /** more **/
    }
  }
}
```

webpack的主要功能都是通过plugin以`this.hooks.xxx.tap('pluginName', fn)`挂载到hooks上，在特定时机执行。
执行时即为`this.hooks.xxx.call('args', callback)`方式，清楚这个模式后，看源码就轻松很多。

# 流程

创建compiler，`WebpackOptionsApply.process` 会根据配置项注册对应的内部插件，compiler.run进入核心流程。
webpack的强大在于在生命周期调用对应的插件处理

关键生命周期：
* entry-options
* compile
* make: 分析入口，创建模块对象
* build-module: 构建模块
* after-compile:
* emit: seal封装。生成assets
* after-emit: render组合输出

翻阅源码时重点查看如下关键函数：

* compile
* make
* compilation.addEntry
* compilation: _addModuleChain
* buildModule（NormalModule.js）
* build - doBuild（NormalModule.js）
* doBuild - runLoaders: 调用loaders, 任何模块都被转成了标准的JS模块
* parse: 获得ast，调用 acorn 对JS代码进行语法分析，然后收集其中的依赖关系
* seal - createChunkAssets - mainTemplate.render, render的renderBootStrap是生成webpack样板代码,template.js的renderChunkModules是真正拼接模块代码字符串的地方。根据之前收集的依赖，决定生成多少文件，每个文件的内容是什么。
* render：这里是生成每个模块的代码片段
* emitAssets: 将各模块的代码片段整合输出到chunk文件中

# 其他一些记录点

## 插件调用时是在创建的临时函数里

调用的插件处理方法都是通过new Function通过字符串拼接生成的。这个函数不止是开发定义的函数，前后还有很多webpack加入的内容，每个函数都不同。

注册插件时，会将要调用的函数先生成好放在柯里化的返回函数里，然后在调用时进入执行，并接收所需参数

![](https://s10.mogucdn.com/mlcdn/c45406/190317_861ld8be6e4j2fb356kkc6gd4jha3_780x396.png)

譬如`this.hooks.make.callAsync(compilation,...`调用时的临时函数有：

![](https://s10.mogucdn.com/mlcdn/c45406/190317_18j04gkh96dh0f1412dkke695e1c6_786x1406.png)

意义何在？我理解是因为多样性，为了给每个插件调用加入不同的上下文执行代码。

## loader

loader是把非js模块处理成js模块

runLoaders返回的result里是含有buffer原数据的数组，经过createSource处理成经过转化后的代码string,
这时候如果loader返回了ast后面将用这个ast，如果没有在经过doBuild回调里的this.parser.parse，调用parse.js中parse函数里的acorn.parse获得ast

## 代码拼接生成

代码分为webpack自己的脚手架代码部分，和经过loader处理的模块代码部分。

render - renderBootstrap处理脚手架部分 - this.hooks.render.call - this.hooks.render.tap("mainTemplate") - this.hooks.module.call - this.hooks.module.tap("JavascriptModulesPlugin")处理模块拼接部分 - emitAssets将拼接的代码输出成对应bundle文件

![](https://s10.mogucdn.com/mlcdn/c45406/190317_50243l55lacacg2d6ffhi8l086hbe_1268x1110.png)

![](https://s10.mogucdn.com/mlcdn/c45406/190317_01k2heagjbl943d8j019dfh593jd4_1772x1064.png)

![](https://s10.mogucdn.com/mlcdn/c45406/190317_4i0ebgi9c2a8ic3k350kjjhg4gd09_1730x1466.png)

![](https://s10.mogucdn.com/mlcdn/c45406/190317_56f4ge5i715ggff225g02k3i419i2_1366x964.png)

![](https://s10.mogucdn.com/mlcdn/c45406/190317_1g04jebfika5b6dgdff98b8ag778e_1194x436.png)

![](https://s10.mogucdn.com/mlcdn/c45406/190317_46388kijljfdg3g858k0f2fj49dc8_980x1042.png)

会看到一些CacheSource利用缓存加速：

![](https://s10.mogucdn.com/mlcdn/c45406/190317_415l8lkg88allfljb1l6c16c7634f_1582x1472.png)

还有ReplaceSource既是处理模块化转化的部分：

![](https://s10.mogucdn.com/mlcdn/c45406/190317_32gji9f5ik1k02eidib6k32c1139j_1730x1428.png)

遍历拼接代码片段:

![](https://s10.mogucdn.com/mlcdn/c45406/190317_24k4i0l2g8chc47gf1ca4ce7l5a3b_1868x972.png)

最后输出到chunk文件：

![](https://s10.mogucdn.com/mlcdn/c45406/190317_3646jci30gg129802eiefb63f2973_1506x982.png)

![](https://s10.mogucdn.com/mlcdn/c45406/190317_8ef9l3a4e1b674fbhh5499280172b_1334x1306.png)

通过以上断点跟踪，就走完了代码拼接到输出的流程。
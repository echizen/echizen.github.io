---
layout: post
title: "webpack-打包后代码分析"
description: "webpack打包后代码分析，webpack模块化，cmd, amd, umd的处理"
category: tech
tags: ['webpack']
---
{% include JB/setup %}

你是否清楚webpack究竟将你的代码处理成了什么样子？webpack是如何实现模块化的？为啥webpack既能支持es6 module又能支持commonjs规范的module？为啥webpack打包出来的模块在commonjs规范的模块里引用时要加default，即`const moduleName = require('modulePath').default`

如果你都清楚，那么本文可以跳过了~

# webpack是什么

官方说: **webpack 是一个现代 JavaScript 应用程序的静态模块打包器(static module bundler)**。浏览器原生是不支持模块化的，虽然有新版本已经开始支持，但是考虑兼容性在相当长的一段时间里我们不能依赖，而webpack帮我们实现了模块化的支持，我们将代码按功能有序的进行模块化组织，webpack将这些模块化的代码合并打包成一个bundle文件，并使用自己的脚手架代码实现了模块化的语义，让模块与模块之间能够作用域隔离，能够互相引用。

webpack比较神奇的是不仅支持js的模块化，还能支持css\file\image等各种文件的模块化，得益于强大的loader将各种类型的文件处理成js模块。

# webpack打包产物分析

## demo

文中相关代码在这里：[https://github.com/echizen/webpack-demo/tree/master/webpack-mod](https://github.com/echizen/webpack-demo/tree/master/webpack-mod)，可以自行clone下来验证

我们通过一个最简单的例子来分析打包产物，创造3个文件：

index.js:

```javascript
import message from './message.js';

export const msg = message
export default msgEntry = message + '!'
```

message.js:

```javascript
import {name} from './name.js';

export default `hello ${name}!`;
```

index.js:

```javascript
export const name = 'world';
```

webpack配置：
```javascript
const path = require('path');

module.exports = {
  mode: "production",
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'index.bundle.js'
  },
  optimization: {
    minimize: false
  }
};
```

看看index.bundle.js的内容：
```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);

// CONCATENATED MODULE: ./src/name.js
const name_name = 'world';
// CONCATENATED MODULE: ./src/message.js


/* harmony default export */ var message = (`hello ${name_name}!`);
// CONCATENATED MODULE: ./src/index.js
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "msg", function() { return msg; });


const msg = message
/* harmony default export */ var src = __webpack_exports__["default"] = (msgEntry = message + '!');

/***/ })
/******/ ]);
```

## 分析

可以看出，webpack打包出来的文件，用installedModules记录缓存的模块分析结果。自定义了一个require的实现`__webpack_require__`来实现模块的依赖引入功能。

这个函数即是实现模块化的重点。原模块被包裹在`function(module, __webpack_exports__, __webpack_require__) {}`函数中。`__webpack_require__`里创造的局部变量module变量来记录此模块函数调用后返回的结果，将`module.exports, module, module.exports, __webpack_require__`作为参数传入，每个模块函数，先调用`__webpack_require__.r(__webpack_exports__)`在`module.exports`上定义`__esModule`属性，作用是和前辈babel转化的结果保持一致，表明这是个由 es6 转换来的 commonjs 输出。。然后执行原模块代码，将export出的内容处理到`module.exports`上，如果是`export default`的内容则直接放置到`module.exports["default"]`上，其他的非default的valName的export通过`__webpack_require__.d`实现的在`module.exports`defineProperty`valName`的getter来获取。这样调用了包裹原模块的函数后，`__webpack_require__`函数最终return出来的`module.exports`上就有我们所有export出来的内容，实现了隔离作用域的模块化。

```javascript

// 调用处
// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
return module.exports;

// 定义处
(function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);

// CONCATENATED MODULE: ./src/name.js
const name_name = 'world';
// CONCATENATED MODULE: ./src/message.js

/* harmony default export */ var message = (`hello ${name_name}!`);
// CONCATENATED MODULE: ./src/index.js
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "msg", function() { return msg; });


const msg = message
/* harmony default export */ var src = __webpack_exports__["default"] = (msgEntry = message + '!');
```

然后还定义了一群常用的功能函数：

- __webpack_require__.d: define getter function for harmony exports，使用Object.defineProperty来定义对象上指定属性的getter函数
- __webpack_require__.r: define __esModule on exports
- __webpack_require__.t: create a fake namespace object，根据不同的mode做不同的处理
- __webpack_require__.n: getDefaultExport function for compatibility with non-harmony modules
- __webpack_require__.o: object.prototype.hasOwnProperty.call

## Scope Hoisting

认真观察你会发现一个现象，按理说我们是3个文件对应的3个模块，但是在打包产物里却被合并成了一个模块。这就是`Scope Hoisting`特性。

webpack3开始引入了Scope Hoisting，Scope Hoisting 的实现原理其实很简单：分析出模块之间的依赖关系，尽可能的把打散的模块合并到一个函数中去，但前提是不能造成代码冗余。 因此只有那些被引用了一次的模块才能被合并。

能使用`Scope Hoisting`特性的要求：
- 必须是ES6规范的模块化
- 没有使用 eval() 函数

modules列表用了Scope Hoisting作用域提升合并成了一个。好处：
- 代码体积更小，因为函数申明语句会产生大量代码；
- 代码在运行时因为创建的函数作用域更少了，内存开销也随之变小。

## commonjs规范的模块打包

上面的示例是es6规范的模块，我们再来看看commonjs规范的模块：

index.js

```javascript
var c = require('./name.js')
exports.default = c
```

name.js

```javascript
let c1 = 'c1'
let c2 = 'c2'
module.exports = {
	c1,
	c2,
}
```

打包产物：

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/

// 省略n个__webpack_require__上挂载的功能函数

/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = 0);
/******/ })
/************************************************************************/
/******/ ([
/* 0 */
/***/ (function(module, exports, __webpack_require__) {

// import message from './message.js';

// export const msg = message
// export default msgEntry = message + '!'

var c = __webpack_require__(1)
exports.default = c


/***/ }),
/* 1 */
/***/ (function(module, exports) {

// export const name = 'world';
let c1 = 'c1'
let c2 = 'c2'
module.exports = {
	c1,
	c2,
}

/***/ })
/******/ ]);
```

可以看到cmd规范的模块没有使用`Scope Hoisting`了，区别也就是从es module都是在操作'module.exports'，变成了如果原模块透出的方式是`module.exports`则继续赋值给`module.exports`的对应属性上，如果原模块使用是`exports.default`则赋值给`module.exports.default`。

**所以webpack通过在`__webpack_require__`上内部构造一个`module`的局部变量，无论原模块是es6还是commonjs，或者是2者混用，都将原透出属性透出到`module.exports`的对应属性上，将透出的`default`属性透出到`module.exports.default`，来达到统一，内部可以支持各种类型的模块通用，因为都已经转化成了commonjs的格式。**

### 为啥commonjs格式的模块引入webpack打包后的模块需要加default？

即`const moduleName = require('modulePath').default`

上面的分析我们知道，commonjs 里的require对应的是webpack内部的`__webpack_require__`透出的`module.exports`，而default的对象被赋值在`module.exports.default`上，default也只是module.exports的一个叫default名的普通透出对象，自然是要加default属性声明的。

那么es6的模块为啥就可以直接`import`而不用管`default`呢？因为规范定义的`import moduleName from 'modulePath'`就是引入一个模块`export default`的内容啊，webpack为了遵循这个规范，在内部对default的内容做了标记处理，在引入的时候直接透出default的内容。

为了查看这个，我们必须打破`Scope Hoisting`特性，来看看import语句和`__webpack_require__`的对应调用关系。于是我们在message.js里使用下eval：

message.js
```javascript
import {name} from './name.js';
eval('var test = 1')
export default `hello ${name}!`;
```

index.js
```javascript
import message from './message.js';

export const msg = message
export default msgEntry = message + '!'
```

打包出来模块相关部分的代码：

```javascript
([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony import */ var _name_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(1);

eval('var test = 1')
/* harmony default export */ __webpack_exports__["a"] = (`hello ${_name_js__WEBPACK_IMPORTED_MODULE_0__[/* name */ "a"]}!`);

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "a", function() { return name; });
const name = 'world';
// let c1 = 'c1'
// let c2 = 'c2'
// module.exports = {
// 	c1,
// 	c2,
// }

/***/ }),
/* 2 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "msg", function() { return msg; });
/* harmony import */ var _message_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(0);


const msg = _message_js__WEBPACK_IMPORTED_MODULE_0__[/* default */ "a"]
/* harmony default export */ __webpack_exports__["default"] = (msgEntry = _message_js__WEBPACK_IMPORTED_MODULE_0__[/* default */ "a"] + '!');

// var c = require('./name.js')
// exports.default = c


/***/ })
/******/ ]);
```

可以看到bundle发生了很大的变化，`export default`的内容不是再赋值给`__webpack_exports__["default"]`，而是赋值给`__webpack_exports__`的`a`, `b`这样内部定义属性的getter上，在`import`语句出处理成`__webpack_require__(moduleId)["a"]`这样找到对应的属性，即es6模块中也没有将default看成特殊属性，而是取了别名，内部也通过别名去`__webpack_require__`找到对应内容，这里的注释`/* default */`在标记时起到与原代码透出变量对应的功能。

为什么要这么搞呢？为啥default的es6和commonjs引入不搞成一致呢？为了跟规范保持一致啊，规范定义的`import moduleName from 'modulePath'`就是引入一个模块`export default`的内容，而`const moduleName = require('modulePath')`获取的是modulePath对应的模块透出的所有内容的一个对象即`module.exports`的内容，`default`只是一个普通属性即`module.exports.default`。

再提个历史，在 babel5 时代，大部分人在用 require 去引用 es6 输出的 default，只是把 default 输出看作是一个模块的默认输出，所以 babel5 对这个逻辑做了 hack，如果一个 es6 模块只有一个 default 输出，那么在转换成 commonjs 的时候也一起赋值给 module.exports，即整个导出对象被赋值了 default 所对应的值。这样就不需要加 default。

babel5的这种做法其实会出问题：

```javascript
// a.js

export default 123;

export const a = 123; // 新增

// b.js 

var foo = require('./a.js');

// 由之前的 输出 123, 变成 { default: 123, a: 123 }，导致使用方要改动引入代码

```

所以babel6不再做`module.exports=exports.default`的处理了，如果因为历史问题依赖这个处理，可以加plugin:`babel-plugin-add-module-exports`

# webpack和babel

webpack和babel都支持es6 module，babel通常作为webpack的loader处理js，这2者又是什么关系呢？

其实webpack2开始已经支持es6 module的处理，如果只是为了模块化，无需在加载babel，只是es6不止module还有很多其他特性需要babel帮我们处理。babel也是将es6 module转化成commonjs的module，从而再经过webpack时当commonjs module进行处理。

譬如, es module:

```javascript
export default 123;

export const a = 123;

const b = 3;
const c = 4;
export { b, c };
```

babel转化后，处理成commonjs格式：
```javascript
exports.default = 123;
exports.a = 123;
exports.b = 3;
exports.c = 4;
exports.__esModule = true;
```

# webpack的打包类型

webpack支持`var、umd、commonjs、commonjs2、amd、amd-require、this、window、global、jsonp`10种类型的打包产物：[output-librarytarget](https://webpack.docschina.org/configuration/output/#output-librarytarget)

### var (默认)

```javascript
(function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
  // module code
  })
])
```

### amd

```javascript
define("modName", [], function() {
  return /******/ (function(modules) { // webpackBootstrap
      // webpackBootstrap
    })([
      // modules code
    ])
})
```

### commonjs2

```javascript
module.exports = (function(modules) { // webpackBootstrap
  // webpackBootstrap
})([
  // modules code
])
```

### umd

`output.library`指定为modName时：
```javascript
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else if(typeof exports === 'object')
		exports["modName"] = factory();
	else
		root["modName"] = factory();
})(window, function() {
  return /******/ (function(modules) { // webpackBootstrap
      // webpackBootstrap
    })([
      // modules code
    ])
})
```

`output.library`未指定时：

```javascript
(function webpackUniversalModuleDefinition(root, factory) {
	if(typeof exports === 'object' && typeof module === 'object')
		module.exports = factory();
	else if(typeof define === 'function' && define.amd)
		define([], factory);
	else {
		var a = factory();
		for(var i in a) (typeof exports === 'object' ? exports : root)[i] = a[i];
	}
})(window, function() {
  return /******/ (function(modules) { // webpackBootstrap
      // webpackBootstrap
    })([
      // modules code
    ])
})
```

### window || global

`output.library`指定为modName时：

```javascript
window["modName"] = (function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
  // modules code
])
```

`output.library`未指定时：

```javascript
(function(e, a) { 
  for(var i in a) e[i] = a[i]; 
}(window, /******/ (function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
    // module code
    })
  ])
))
```

### this

`output.library`指定为modName时：

```javascript
this["modName"] = (function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
  // modules code
])
```

`output.library`未指定时：

```javascript
(function(e, a) { 
  for(var i in a) e[i] = a[i]; 
}(this, /******/ (function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
    // module code
    })
  ])
))
```

### commonjs

`output.library`指定为modName时：

```javascript
exports["modName"] = (function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
  // modules code
])
```

`output.library`未指定时：

```javascript
(function(e, a) { 
  for(var i in a) e[i] = a[i]; 
}(exports, /******/ (function(modules) { // webpackBootstrap
  // __webpack_require__ 相关定义
})([
/* 0 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
    // module code
    })
  ])
))
```

可以看到各种模块化的方式都是使用对应规范的代码包裹了webpack原本的脚手架代码和模块加载代码，使执行后的结果挂载到入`windows`、`module.exports`、`exports["modName"]`这些对象上。



# 参考文档

自己倒腾了demo之后，去网上搜了下，发现这位同僚的文章讲的更透彻：

[import、require、export、module.exports 混合使用详解](https://juejin.im/post/5a2e5f0851882575d42f5609)

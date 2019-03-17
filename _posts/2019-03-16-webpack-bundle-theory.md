---
layout: post
title: "webpack-打包原理"
description: "webpack打包原理"
category: tech
tags: ['webpack']
---
{% include JB/setup %}

开始填坑系列，将前2个月原理&源码维度学习的webpack\vue\react所得收获整理出来。

webpack的打包原理是怎样，其实刨除loader\plugin的核心功能实现起来并不难，这份仓库代码很好的展现了核心逻辑：[https://github.com/ronami/minipack](https://github.com/ronami/minipack)

简要代码：

```javascript
(function(modules) {
  function require(id) {
    const [fn, mapping] = modules[id];

    function localRequire(name) {
      return require(mapping[name]);
    }

    const module = { exports: {} };

    fn(localRequire, module, module.exports);

    return module.exports;
  }

  require(0);
})({
  0: [
    function(require, module, exports) {
      "use strict";

      var _message = require("./message.js");

      var _message2 = _interopRequireDefault(_message);

      function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

      console.log(_message2.default);
    }
    , { "./message.js": 1 }
  , ]
  , 1: [
    function(require, module, exports) {
      "use strict";

      Object.defineProperty(exports, "__esModule", {
        value: true
      });

      var _name = require("./name.js");

      exports.default = "hello " + _name.name + "!";
    }
    , { "./name.js": 2 }
  , ]
  , 2: [
    function(require, module, exports) {
      "use strict";

      Object.defineProperty(exports, "__esModule", {
        value: true
      });
      var name = exports.name = 'world';
    }
    , {}
  , ]
, })

```

打包过程：

- 提供一个入口文件
- babylon分析这个入口文件
    - 提取依赖
    - 获得babel转成es5后的兼容代码
    - 确定个全局标识符id
- 再对入口文件的依赖进行递归处理，进行上述分析，得到依赖图graph
- 生成bundle
  - 将graph的每个片段都插入一些代码，成为 { id: [func(require, module, exports){ origin code}, dependenceMap]}的结构
  - 定义自己的依赖加载函数require
  - 将两部分字符串代码片段合成

核心是打包工具自己实现的依赖加载器require，require里创造的module变量来记录fn调用后返回的结果，babel转完后，如果有返回值会挂到exports的变量上，着这里被module.exports接收，并作为require函数的返回，所以require函数的返回是执行对应的文件后所获得的该文件export出来的内容，localRequire里将这个值返回出来也就是原文件import 需要得到的值，供原文件引用。
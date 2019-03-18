---
layout: post
title: "webpack-loader与plugin简析"
description: "webpack loader与plugin的原理和实现"
category: tech
tags: ['webpack']
---
{% include JB/setup %}

本文浅度预警，只是loader和plugin作为webpack的一个重要内容，应该作为一个章节单独理理。

# loader

loader的核心作用就是把非标准js模块转化为js模块，这些非js模块包括css、file以及vue、jsx这样的模块，供后续webpack实现模块化处理并打成bundle文件。

webpack的loader设计是如此有魅力，正式因为它的存在，我们才能all in js，前端工程化之路才如此规模化。

来看看几个典型loader的处理方式：

## file-loader

从最简单的开始，file-loader没有对原文件做任何处理。会根据文件内容生成hash值（或配置其他规则生成一个唯一值），将这个值作为资源路径名url，通过`emitFile(url, content)`放到 public path 目录下，将`return 'module.exports = __webpack_public_path__ + '+ JSON.stringify(url)`, 这样其他文件里引用这个file的地方就会使用生成的资源路径了。

[file-loader代码](https://github.com/webpack-contrib/file-loader/blob/master/src/index.js)

## url-loader

url-loader一般用来处理将小size的图片资源转化为base64编码嵌入到代码内。

所以url-loader 并不复制文件，而是把文件做base64编码，直接嵌入到CSS/JS/HTML代码中：
- 获取 limit 参数
- 如果 文件大小在 limit 之内，则直接返回文件的 base64 编码后内容
- 如果超过了 limit ，则调用 `file-loader`处理

[url-loader代码](https://github.com/webpack-contrib/url-loader/blob/master/src/index.js)

## css-loader与style-loader

css-loader和style-loader就复杂一些了。

一般css-loader和style-loader都是配套使用，代在经过style-loader处理期间会调用css-loader。

css-loader 的作用是处理css中的 @import 和 url 这样的外部资源，替换成require。转化过程中将本文件的代码以及本文件的依赖都送入到exports的数组里，并生成`exports.locals`含有唯一标识符的map来支持css module映射表。最终将css文件内容处理成可以require的模块。

style-loader 相对好理解一点，作用是把样式插入到 DOM中，方法是在head中插入一个style标签，并把样式写入到这个标签的 innerHTML 里

借用一下别人的图：

![image](https://github.com/lihongxun945/diving-into-webpack/raw/master/images/style-loader-and-css-loader-pipeline.png)

[css-loader代码](https://github.com/webpack-contrib/css-loader)
[style-loader代码](https://github.com/webpack-contrib/style-loader)

## 开发loader

开发loader的姿势倒是非常简单，

同步loader：

```javascript
module.exports = function(content, map, meta) {
  return someSyncOperation(content);
}
```

同步loader返回多个结果：

```javascript
module.exports = function(content, map, meta) {
  this.callback(null, someSyncOperation(content), map, meta);
  return; 
}
```

异步loader：

```javascript
module.exports = function(content, map, meta) {
  var callback = this.async();
  someAsyncOperation(content, function(err, result) {
    if (err) return callback(err);
    callback(null, result, map, meta);
  });
};
```

详情：

- [loader API](https://webpack.docschina.org/api/loaders/)
- [编写一个 loader](https://webpack.docschina.org/contribute/writing-a-loader/)

# plugin

> 插件向第三方开发者提供了 webpack 引擎中完整的能力。使用阶段式的构建回调，开发者可以引入它们自己的行为到 webpack 构建流程中

插件能在构建的各个阶段插入自己的行为，能影响改变构建输出内容，拆分增加构建文件等等

一个插件由以下构成

- 一个具名 JavaScript 函数。
- 在它的原型上定义 apply 方法。
- 指定一个触及到 webpack 本身的 事件钩子。
- 操作 webpack 内部的实例特定数据。
- 在实现功能后调用 webpack 提供的 callback。

```javascript
// 一个 JavaScript class
class MyExampleWebpackPlugin {
  // 将 `apply` 定义为其原型方法，此方法以 compiler 作为参数
  apply(compiler) {
    // 指定要附加到的事件钩子函数
    compiler.hooks.emit.tapAsync(
      'MyExampleWebpackPlugin',
      (compilation, callback) => {
        console.log('This is an example plugin!');
        console.log('Here’s the `compilation` object which represents a single build of assets:', compilation);

        // 使用 webpack 提供的 plugin API 操作构建结果
        compilation.addModule(/* ... */);

        callback();
      }
    );
  }
}
```

我们来写一个简单的示例插件，生成一个叫做 filelist.md 的新文件；文件内容是所有构建生成的文件的列表。这个插件大概像下面这样：

```javascript
class FileListPlugin {
  apply(compiler) {
    // emit 是异步 hook，使用 tapAsync 触及它，还可以使用 tapPromise/tap(同步)
    compiler.hooks.emit.tapAsync('FileListPlugin', (compilation, callback) => {
      // 在生成文件中，创建一个头部字符串：
      var filelist = 'In this build:\n\n';

      // 遍历所有编译过的资源文件，
      // 对于每个文件名称，都添加一行内容。
      for (var filename in compilation.assets) {
        filelist += '- ' + filename + '\n';
      }

      // 将这个列表作为一个新的文件资源，插入到 webpack 构建中：
      compilation.assets['filelist.md'] = {
        source: function() {
          return filelist;
        },
        size: function() {
          return filelist.length;
        }
      };

      callback();
    });
  }
}

module.exports = FileListPlugin;
```

更多参考：

- [编写一个插件](https://webpack.docschina.org/contribute/writing-a-plugin/)
- [Plugin API](https://webpack.docschina.org/api/plugins/)

# 参考文献

作者写的真的很不错，他的系列是webpack官网资料外我作为webpack源码阅读的指导文章，本文也是大量引用了以下文章内的片段。

- [webpack 源码解析三：详解 style-loader 和 css-loader](https://github.com/lihongxun945/diving-into-webpack/blob/master/3-style-loader-and-css-loader.md)
- [webpack 源码解析 四：file-loader 和 url-loader](https://github.com/lihongxun945/diving-into-webpack/blob/master/4-file-loader-and-url-loader.md)


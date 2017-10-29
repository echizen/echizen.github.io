---
layout: post
title: "开发postcss插件"
description: "开发postcss插件, css ast, css抽象语法树"
category: tech
tags: [js, css, tool]
---
{% include JB/setup %}

有时候，我们有处理css的需求，可能需要个工具来转化我们的css代码，譬如把正常的移动端web css代码里的rem、px单位转化成小程序里的rpx单位。

这时候写个postcss插件是个不错的选择。

### css ast

postCss会帮我们分析出css的抽象语法树，要想写插件去处理css，需要先了解一下它透出的css ast。

#### 类型

css ast主要有3种父类型

- AtRule: @xxx的这种类型，如@screen
- Comment: 注释
- Rule: 普通的css规则

还有几个个比较重要的子类型：

- decl： 指的是每条具体的css规则
- rule：作用于某个选择器上的css规则集合

初次接触，可以去这里看看ast�树结构，会清晰不少：

[http://astexplorer.net/#/2uBU1BLuJ1](http://astexplorer.net/#/2uBU1BLuJ1)

![](http://s3.mogucdn.com/mlcdn/c45406/171029_0ak3ehbbke8b5e5bfk5f56f6574kj_2204x1164.png)

![](http://s11.mogucdn.com/mlcdn/c45406/171029_539a04idg9af916g4i5gi57fb7ibb_2026x1124.png)

一个ast节点基本有如下信息：

- nodes: css规则的节点信息集合
    - decl: 每条css规则的节点信息
    - prop: 样式名，如`width`
    - value: 样式值， 如`10px`
- type: 类型
- source: 包括start和end的位置信息，start和end里都有line和column表示行和列
- selector: type为rule时的选择器
- name: type为`atRule`时@紧接rule名，譬如`@import 'xxx.css'`中的`import`
- params: type为`atRule`时@紧接rule名后的值，譬如`@import 'xxx.css'`中的`xxx.css`
- text: type为`comment`时的注释内容


### postCss操作方法

postCss为我们提供了一些方便的操作方法

#### 遍历

- walk: 遍历所有节点信息，无论是atRule、rule、comment的父类型，还是`rule`、 `decl`的子类型
- walkAtRules：遍历所有的atRule
- walkComments
- walkDecls
- walkRules

这些方法都透出了遍历对象的参数：

    root.walkDecls(decl => {
        decl.prop = decl.prop.split('').reverse().join('');
    });
    
#### 处理

如同jquery给出了很多操作dom节点的方法一样，postCss给出了很多操作css规则的方法，每种类型的节点稍微不同，如root节点的

![](http://s3.mogucdn.com/mlcdn/c45406/171029_8d7kllad227b7hl5ekjjkg801l43k_248x952.png)

### 处理css

搞清楚了postcss为我们提供的css ast结构和提供的操作函数后一切就好办了。无非是找到我们要处理的节点，对于要处理的内容做一些增减或者替换操作。

处理css的方式其实有2种：编写postcss plugin，如果你的操作非常简单也可以直接利用postcss.parse方法拿到css ast后分析处理

#### postcss plugin

postcss插件如同babel插件一样，有固定的格式

    export default postcss.plugin('postcss-plugin-name', function (opts) {
      opts = opts || {};
      return function (root, result) {
        // 处理逻辑
      };
    });
    
注册个插件名，并获取插件配置参数opts

返回值是个函数，这个函数主题是你的处理逻辑，有2个参数，一个是`root`，css ast的根节点。另一个是result，返回结果对象，譬如`result.css`，获得处理结果的css字符串，`result.message`包含组件里创建的warnings和自定义信息，`result.warn()`创造一个warning实例并将其加入到`result.message`中，`result.warnings()`获得所有创造过的warning。

注意组件的异常信息处理，不要直接`console`，因为一些 PostCSS 处理器会过滤掉`console`的输出导致异常信息丢失，用`node.warn`或者`node.error`创造`CssSyntaxError `实例，会自动带上源码中的位置信息帮助debug，加到异常信息队列里。

#### 直接调用postcss命名空间下的方法

可以用`postcss.parse`来处理一段css文本，拿到css ast，然后进行处理，再通过调用`toResult().css`拿到处理后的css输出

### 示例

写了个web页面的rem\px单位转成小程序wxss里的rpx单位的小插件示例：

[https://github.com/echizen/postcss-unit2rpx/blob/master/index.js](https://github.com/echizen/postcss-unit2rpx/blob/master/index.js)
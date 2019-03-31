---
layout: post
title: "react-导读"
description: "react你该了解的内容，看react源码前应该了解的内容"
category: tech
tags: ['react']
---
{% include JB/setup %}

看react源码前可以先这些资料，帮助抓住整体思路，了解设计。

# 设计思想
[https://reactjs.org/docs/design-principles.html](https://reactjs.org/docs/design-principles.html)

讲了符合组件的设计和意义，讲了什么场景下的内容会被抽象到react框架内部实现，特别是`Scheduling`章节好好的解释了一波react声明式组件的优势，可以将组件更新流程交给框架层来调度，也就让框架可以优化处理，譬如延时、合并更新这些，也是fiber算法 暂停重启更新、等待浏览器空闲时片段式更新 这种做法的基础，总之这个介绍让我豁然开朗，发现了声明式的魅力并在自己的一些开发设计中用上。

# 核心模型
[https://github.com/reactjs/react-basic#transformation](https://github.com/reactjs/react-basic#transformation)

介绍了对dom节点的element抽象、符合组件的模型、state状态模型等。


# 核心实现
[https://reactjs.org/docs/implementation-notes.html](https://reactjs.org/docs/implementation-notes.html)

react16以前的设计，解释了各种组件类型的实现、渲染、更新实现等。


# fiber概念
[https://github.com/acdlite/react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)

介绍了fiber架构设计，协同和更新2个阶段，scheduling和work部分，fiber节点的构成等重要概念，不过没有连成整体。

# react Components,Elements,Instances
[https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)

介绍了`Components`, `Elements`, 和 `Instances`这3个基础元素的含义和实现以及联系。


# 从0实现react系列

从0实现向来是抓住整体思路和设计的不二法宝

[React 源码分析，实现一个简易版的React](http://react-china.org/t/react-react/26788)
[https://github.com/hujiulong/blog](https://github.com/hujiulong/blog)

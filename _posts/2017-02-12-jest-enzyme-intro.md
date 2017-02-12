---
layout: post
title: "使用jest+enzyme进行react项目测试 - 介绍篇"
description: "使用jest+enzyme进行react项目测试, jest+enzyme介绍, jest+enzyme选型优势, jest+enzyme使用场景"
category: tech
tags: [测试, react]
---
{% include JB/setup %}

在真正了解到jest和enzyme之前，选择jest作为我react项目的测试框架，仅仅从以下几个方面：

- facebook开发，他们自己在用，经得起考验，质量有保障
- 社区活跃，出了问题应该容易找解决方案
- antd用的也是这个，这样我可以参考一下人家怎么写的: 后来发现是antd的测试基本只是snapshot这方面的。。。

但是了解和使用jest+enzyme之后，从未接触过测试相关知识的我仿佛打开了新世界的大门~

## jest

jest 作为一款测试框架，拥有测试框架该有的一套体系，丰富的断言库，大多数api与老牌的测试框架如jasmine、mocha，譬如常用的expect、test(it)、toBe等，都非常好用。内部也是使用了jasmine作为基础，在其上封装。但是因为Snapshot这个特色功能，非常适合react项目的测试。

### 特色

- Snapshot组件快照
    
    借用`react-test-renderer`库的`renderer`获得react组件渲染成的React树，调用toJSON接口格式化，在使用`jest`的`expect(tree).toMatchSnapshot()`将快照与上一次的快照作对比，首次生成的某个测试案例的快照将会被保存下来，以后每次运行时都会与上一次对比，如果发现不匹配会抛出错误，需要自己去查看差异是否是合法的需要更新的内容。

    这种方式对比react组件渲染后的内容，在静态ui上，非常高效的找出差别，维护稳定性。
    
    
- 多线程同时跑测试案例，速度快

    这个我观察了一下我写的不多的几个测试，5个测试脚本一起跑和只跑1个时间基本相同，5s左右，环境初始化还是挺慢的，看单个脚本的分析，每个测试单元都只耗了几百毫秒，环境的初始化时秒级别的。但是确实是多线程同时跑速度大大提升。
    
- 可配置的coverage reports

    生成测试覆盖报告无需再下其他依赖，已被内置，`jest --coverage`就能生成coverage文件夹保存，方便分析。形如：
    
    ![https://s10.mogucdn.com/p2/170211/114994688_0kj0fjh6gb5c6j79i1bf95l6d244a_2432x1348.png](https://s10.mogucdn.com/p2/170211/114994688_0kj0fjh6gb5c6j79i1bf95l6d244a_2432x1348.png)
    
    
- 强大的mocking库

 可以mock值、函数、文件多种类型
 
    - mock函数：mock的函数可以被追踪到，`const handler = jest.fn()`，譬如后续可以通过`expect(handler).toBeCalled()`检查mock的函数是否如期被调用，`expect(handler).toBeCalledWith('arg')`检查mock的函数是否调用时传入的参数是`arg`等。
    - mock文件：诸如css、image等于逻辑测试无关的资源文件可以在配置时就被mock掉`"\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/spec/__mocks__/fileMock.js",
      "\\.(css|scss)$": "<rootDir>/spec/__mocks__/styleMock.js",`，这样`import`时就不会真的去引入这些无用的文件了。
    - time mock：`setTimeout, setInterval, clearTimeout, clearInterval`默认是被mock的，这样就不会再执行是真的等待这些函数获取的时间参数，影响测试执行速度了，但是内部特殊的mock让其依旧能保证异步进行和执行顺序。
    
- 内置jsdom，提供了dom依存的环境
    
    jest 配合enzyme,enzyme可以在jsDom里渲染出虚拟dom，然后我们可以操作它，进行交互测试。依旧有window、document等对象，但是无法往这个dom中插入script标签进行其他资源文件的加载。

## enzyme

enzyme提供了几种方式中将react组件渲染成真实的dom，提供了类似jquery的api来获取dom；提供了simulate函数来模拟事件触发；提供接口让我们获取到组件的state和props并且能对其进行操作。

enzyme其实是`react-test-renderer`的封装，`react-test-renderer`的api非常不友好，但是enzyme开发的api跟jquery一致。如：

![](http://s2.mogucdn.com/p2/170211/114994688_189fl06338afj5k0fg49e7g86iha4_548x1300.png)

还可以接入`enzyme-to-json`，这样在将enzyme生成的react树生成snapshot时就可以使用`toJson`进行格式化，提高生成的snapshot的可读性，代替了`react-test-renderer`的`toJSON`接口。
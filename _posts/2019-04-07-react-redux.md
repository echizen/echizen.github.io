---
layout: post
title: "react和redux生态源码解析"
description: "react和redux生态源码解析, redux 源码及作用, redux-thunk源码及作用，react-redux源码及作用"
category: tech
tags: ['react', 'redux']
---
{% include JB/setup %}

redux是我阅读知名库源码的迈开的第一步，2年前结合函数式编程思想也总结过redux源码阅读笔记。但是当时没有将整个体系串起来弄清楚。现在再读，redux简洁明了的源码真是感人，整个流程特别容易理解了。

# redux

核心能力：提供数据存储，分发、变更的机制。意在管控数据流的变更，让一切变更可感知，配合单向数据流。

包含`createStore、applyMiddleware、bindActionCreator、combineReducer`四个组成部分，也是对外提供的api。其中`bindActionCreator`、`combineReducer`为辅助工具。

## createStore

核心的`createStore`提供获取state的`getState`方法和变更数据的`dispatch`方法，`applyMiddleware`提供的中间件组装机制，能让中间件在数据变更前进行对应处理，这个阶段可以更改state，中间件可以封装出更有能力的dispatch。

- `dispatch`的能力是调用`reducer`函数传入`oldState`和`action`获得最新的`state`, 并且调用`store.subscribe`注册的`listeners`。
- getState则直接返回createStore里的局部变量currentState

如果有enhancer如调用中间件的`applyMiddleware`，则将当前函数作为参数传给`applyMiddleware`处理。否则，构造出`getState\dispatch`返回

![](https://s10.mogucdn.com/mlcdn/c45406/190407_8f77e43g8ihabkhd7dkh427e485a1_1016x424.png)

## applyMiddleware

真正调用createStore获得dispatch和getState，给每个中间件注入这2个接口

### dispatch的封装

```javascript
dispatch: function dispatch(action){
  return _dispatch(action);
}
```

这种写法是为了保证是middleware第一层获取到的dispatch是最初来自createStore的的dispatch，而后面`_dispatch=compose.apply(undefined, chain)(store.dispatch)`的dispatch则是被中间件包装处理的含有各种功能的dispatch。而这一步正是调用下面redux thunk中`createThunkMiddleware`封装的dispatch的地方:

![](https://s10.mogucdn.com/mlcdn/c45406/190407_3ga706ke2f6c8b7l483dk44dliee0_914x404.png)
可以看出next就是compose组合的各个中间件函数。

### redux中间件

中间件可以接受dispatch和getState函数，且必须返回action, 格式：

```javascript
function middleware(){
    return ({dispatch, getState}) => next => action => {
        //封装action
        return action
    }
}
```

这个函数式的经典风格其实挺绕的，第一层是在applyMiddleware里调用`middleware(middlewareAPI)`消费，第二层next是applyMiddleware的compose获取封装的dispatch时消费，第三层action是最终action触发dipatch调用时消费。

### applyMiddleware的调用时机

createStore的调用有2种格式：

- ![](https://s10.mogucdn.com/mlcdn/c45406/190407_3heb64b1gfglli84k4c0fbake2icj_818x270.png)

- ![](https://s10.mogucdn.com/mlcdn/c45406/190407_6dh8fkbb95g6g0hliig76hi8gl4l4_810x190.png)

无论哪一种，最终经过内部逻辑后都是一样的后续流程。最后一个参数就是enhancer，最终会调用`enhancer(createStore)(reducer, preloadedState)`

所以`applyMiddleware`里的第一层`return (createStore) => {}`在这里调用，第二层也紧接着被调用。

可以看出，applyMiddleware作用是获取各个中间件封装后的最终dispatch并和原始store一起返回。

![](https://s10.mogucdn.com/mlcdn/c45406/190407_42j9gieh2cdf7818k6kehgh48ab6f_1660x702.png)

## bindActionCreator

调用姿势：

```javascript
connect(
  (state) => {
    return {
      compName :state.compNameState,
    }
  },
  (dispatch) => {
    return {
      actions: bindActionCreators(compNameAction, dispatch)
    }
  }
)(comp, compNameAction);
```

这里`connect`来源于`react-redux`, `bindActionCreators`来源于redux。

 将action函数用dispatch封装一层，视图层就不需要再传入dispatch了。bindActionCreators中的dispatch来源于视图层`<provider>`中注入的store里的dispatch

![](https://s10.mogucdn.com/mlcdn/c45406/190407_1e1lh0h5026jcb92l4l5ld3egfc6g_922x166.png)

# redux-thunk

为了解决频繁出现的异步action的需求，`redux-thunk`也是必不可少的库

![](https://s10.mogucdn.com/mlcdn/c45406/190407_79le1kej1he53aidfa7ilc7d4c4k5_932x414.png)

当有异步等需要处理时，我们写aciton时格式是：

```javascript
function actionFn(){
    return (dispatch, getstate) => {
        return dispatch(otherFn)
    }
}
```

这样就能将`redux-thunk`注册的`dispatch`和`getState`给内部层层调用，直到最终dipatch获得的是简单的对象型action，走next(action)。这里的next如果有中间件，会进入中间件里。

![](https://s10.mogucdn.com/mlcdn/c45406/190407_68l0j0g3jhg2c0043c08hjd48b13d_1080x360.png)

中间件里的next指向下一个中间件:`store => next => action => { //handler }`

# react-redux

可以看到redux虽然总是被用在react的生态体系内，但是其实他们没有绑定关系，redux完全是个独立的管理数据的库。要和react配套使用，要`react-redux`来作为连接桥梁。

理解了这个库的作用才理解了react和redux那一套是怎么闭环的。

它的实现是将redux store里记录的state 注入到组件里，通过提供`Provider`和高阶组件`Connect`来连接。

## Provider

Provider仅仅是获取到createStore创建出的store对象供自己调用，再传递到内部组件，实现让开发者写的组件能通过props获取到redux管理的数据。

![](https://s10.mogucdn.com/mlcdn/c45406/190407_61e0dg7j88dgl1i5ih29cacd1ijhk_470x274.png)

## Connect:

通过调用`store.subcribe`来监听state的变更，在state变更时对自己的组件`setState`，从而达到更新视图获取组件的state
还会有`mapStateToProps`、`mapDispatchToProps`来分发只记录被过滤的reducer和action作为当前组件的state
其他一坨判断都是更新之类的把控

# 参考文档

[Redux入坑进阶-源码解析](https://github.com/ecmadao/Coding-Guide/blob/master/Notes/React/Redux/Redux%E5%85%A5%E5%9D%91%E8%BF%9B%E9%98%B6-%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
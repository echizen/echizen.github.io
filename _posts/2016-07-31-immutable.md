---
layout: post
title: "Immutable试用"
description: "Immutable的优缺点"
category: tech
tags: [js,react,redux]
---
{% include JB/setup %}

之前有把一个项目想接入Immutable，试用了一下，谈点想法。改造了1天半，最终放弃了改造，因为改造成本实在太高太高了。。。

# Immutable的适应

使用Immutable，除了要清楚他定义的数据都是是不可变数据类型。还要清楚Immutable定义的数据类型与js的数据类型不一样。

Immutable拥有：`List, Stack, Map, OrderedMap, Set, OrderedSet, Record`这几种数据类型。List数据部分结构与js中的array结构类似，map与object类似。

可以使用`formJs()`接口将js格式的数据很方便转成Immutable中的类型，Immutable中的数据也可以通过`toJs()、toArray()、toObject`这些接口很方便的转成js数据类型的数据。formJs是个深层转换，会将你原本js数据结构中的每一层都转成相应的Immutable类型。

	let jsObj = {
		name:'Immutable',
		structure:[List, Stack, Map, OrderedMap, Set, OrderedSet, Record]
	}
	
	let ImuData = formJs(jsObj);
	
得到的结构是：

	Map{
		name:'Immutable',
		structure: List [List, Stack, Map, OrderedMap, Set, OrderedSet, Record]
	}


# Immutable的诱惑

如果读react官方文档，会看到很多地方都在推荐使用Immutable，Immutable是facebook力荐的一个库，号称与react组合使用能提升10倍性能，github star数至16.7.31已至14000+了。

由于react要求我们不要改变原始state，所有数据的变更都通过setState(newstateObj)的方式，redux也是要求不改变原始state，所有对数据的更新都放到reducer中来做。但是实际生产中，处处注意深拷贝以不改变原始数据实在太麻烦了。

Immutable的所有数据都是不可改变型，而且提供了一系列接口供我们创建数据、更新数据、合并数据，每次操作的结果都是个新数据，不修改源数据。这些接口也非常方便使用，譬如合并数据和更新数据：

	var nested = Immutable.fromJS({a:{b:{c:[3,4,5]}}});
	// Map { a: Map { b: Map { c: List [ 3, 4, 5 ] } } }
	var nested2 = nested.mergeDeep({a:{b:{d:6}}});
	// Map { a: Map { b: Map { c: List [ 3, 4, 5 ], d: 6 } } }
	var nested3 = nested2.updateIn(['a', 'b', 'd'], value => value + 1);
	// Map { a: Map { b: Map { c: List [ 3, 4, 5 ], d: 7 } } }
	
	var nested4 = nested3.updateIn(['a', 'b', 'c'], list => list.push(6));
	// Map { a: Map { b: Map { c: List [ 3, 4, 5, 6 ], d: 7 } } }
	
判断数据是否相等用的也是`is()`接口。

还提供了Seq这种懒类型数据，下雨消耗性能的链式查找方法例如map、fileter，Seq不保存中间层数据，对最终结果没有影响的数据都被忽略，非常高效。

# Immutable在react+redux结构的项目中的使用

## combineReducers

combineReducers接受的参数和返回的产物都是object，而Immutable作用的reducer不是对象，所以需要包装，可以使用`redux-immutable`库。

	// store.js
	import { createStore, applyMiddleware } from 'redux';
	import thunkMiddleware from 'redux-thunk';
	import createLogger from 'redux-logger';
	
	import { combineReducers } from 'redux-immutable';
	import bookList from '../reducers/bookList';
	import bookDes from '../reducers/bookDes';		
	
	const rootReducer =  combineReducers({
		bookList,
		bookDes
	});
	
	const logger = createLogger();
	const createStoreWithMiddleware = applyMiddleware(
	  thunkMiddleware,
	  logger
	)(createStore);
	
	const configureStore = configureStore(initialState) {
	  const store = createStoreWithMiddleware(rootReducer, initialState);
	  return store;
	}
	
	
	// index.js
	import React from 'react';
	import { render } from 'react-dom';
	import { Router, browserHistory } from 'react-router';
	import { Provider } from 'react-redux';
	
	import routes from './routes';
	import configureStore from './store';
	
	
	const store = configureStore();
	
	render((
	  <Provider store={store}>
	      <Router history={browserHistory}>
	        {routes}
	      </Router>
	  </Provider>
	), document.getElementById('app'));
	
由于`connect`的state参数能接受的也是纯object，所以传给connnect的state需要外包一层，使用`{ item:Map itemContent }`的结构。

	let connectRedux = function(component, model, actions = null) {
	  return connect(
	    (state) => {
	      let needState = {};
	      needState[model] = state.get(model);	      
	      return needState;
	    },
	    actions ? (dispatch) => {
	      return {
	        actions:bindActionCreators(Object.assign({},actions), dispatch)
	      } 
	    }:null
	  )(component);
	}
	
在reducer中：

	import Immutable from 'immutable';
	const { Map, List, fromJS } = Immutable;
	import * as ActionTypes from '../constants/ActionTypes'
	const initialState = fromJS({
	  name:'book',
		content:{
			section:[
				{
					title:'first'
				},{
					title:'second'
				}
			],
			page:200
		}
	});
	
	export default function(state = initialState, action) {
	  switch (action.type) {
	    case ActionTypes.CHANGE_NAME:
	        return state.update(['content','page'],value => action.page);
	
	    case ActionTypes.ERROR:
	    default:
	        return state;
	  }
	}
	
然后譬如你的bookDes component获得props格式为：

	bookDes:{
		name:'book',
		content:Map {
			section:List [
				Map {
					title:'first'
				},
				Map {
					title:'second'
				}
			],
			page:200
		}
	}

获取prop下的Immutable类型的数据需要通过get接口，例如获取name需要`this.props.bookDes.get('name')`;获取section则要`this.props.bookDes.get('content').get('section)`。

这种接口用起来挺麻烦的，然后所有跟state相关的地方，只要你使用了Immutable类型的数据，就需要使用它的接口获得数据、操作数据，它的语法还是挺不友好的。但是也没办法，毕竟改变了数据类型，且没办法覆盖重写js的`.`操作符，不然会带来更多副作用。我去看了lodash的接口也是类似。

# Immutable的痛点

最终我在尝试后还是放弃了改造，因为成本实在太大，所有数据操作的地方，因为数据类型变了，都得换用Immutable的接口重写。

Immutable与react、redux并没有完美融合，刚嵌入时你会看见无数的`Uncaught Error: Invariant Violation: mapStateToProps must return an object`，不能将Immutable类型的数据直接传给`mapStateToProps`挂载到props上，所以还得将Immutable类型的state放置到对象中作为属性值。

项目后期接入非常麻烦，改造成本太大。

但是Immutable在标准化和性能方面都很有优势，并不能因为接口不友好就被放弃使用，只是建议在项目开始阶段就接入。

# 黄金外链

[https://github.com/facebook/immutable-js/](https://github.com/facebook/immutable-js/)

[https://facebook.github.io/immutable-js/docs/#/](https://facebook.github.io/immutable-js/docs/#/)


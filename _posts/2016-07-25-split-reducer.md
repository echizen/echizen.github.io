---
layout: post
title: "打入react&redux（3）- reducer的拆分与组合"
description: "reducer的拆分与组合，reducer的划分原则"
category: tech
tags: [redux]
---
{% include JB/setup %}

react机制下一切皆组件，没有页面的概念。但是在使用redux划分state时还是有页面倾向的，说页面可能并不准确，更准确的是state是按功能集合划分的。

# reducer中state的划分

state总体上是按页面划分的，譬如现在有个列表页和详情页，还有一些用于多个组件的公用信息如个人信息（或者嵌入多个页面不同地方的组件，总之是公用信息）。我会这么划分：

## 在各页面的reducer中维护各自的state

- list页的reducer:
	
		// list.js
		import * as ActionTypes from '../constants/ActionTypes';
		const initialState = {
		    totalPage: 1,
		    pageIndex: 0, 
		    list:[]
		};
		
		export default function handler(state = initialState, action) {
		    switch (action.type) {
		        case ActionTypes.APPEND_LIST:
		            //...
		        default:
		            return state;
		    }
		}
	
	
- detail页的reducer:

		//detail.js
		import * as ActionTypes from '../constants/ActionTypes';
		const initialState = {
		    title: 1,
		    author: 0, 
		    content:'',
		    comment:[]
		};
		
		export default function handler(state = initialState, action) {
		    switch (action.type) {
		        case ActionTypes.APPEND_COMMENT:
		            //...
		        default:
		            return state;
		    }
		}

- userInfo组件的reducer:

		//userInfo.js
		import * as ActionTypes from '../constants/ActionTypes';
		const initialState = {
		    userName:'',
		    userId:0,
		    avator:''
		};
		
		export default function handler(state = initialState, action) {
		    switch (action.type) {
		        case ActionTypes.GET_INFO:
		            //...
		        default:
		            return state;
		    }
		}

## 使用combineReducers将state合并
	
	// reducers.js
	import { combineReducers } from 'redux';
	
	import list from '../reducers/list';
	import detail from '../reducers/detail';
	import userInfo from '../reducers/userInfo';
	
	export default combineReducers({
		list,
		detail,
	    userInfo
	});
	
	
## connect redux

封装一个connectRedux函数将state的分支按需分发给组件。

	import React from 'react';
	import { connect } from 'react-redux';
	
	/* components */
	var List = require('ROOT_PATH/pages/ListView');
	var Detail = require('ROOT_PATH/pages/DetalView');
	
	var connectRedux = function(component, model = [], actions = null) {
	  return connect(
	    (state) => {
	      let needState = {};
	      if(model.length){
	        model.map((item)=>{
	          Object.assign(needState, state[item]);
	        })
	      }else{
	        needState = state;
	      }
	      return needState;
	    }
	  )(component);
	}
	
	List = connectRedux(List, ['list','userInfo']);
	Detail = connectRedux(Detail, ['detail','userInfo']);
	
这样List组件能访问到：

	const {totalPage, pageIndex, list, userName, userId, avator} = this.props
	
Detail组件同理。


# action的处理

项目中会遇到这种情景。用户的某个操作可能导致需要更新属于state不同部分的数据。依旧拿上文中的数据举例，例如用户列表包括自己，假想有个弹窗可以更改自己头像，用户的更改将导致`state.list[0].avator`和`state.userInfo.avator`都被更新，我们该如何派发action更新2部分数据呢？

## 不能同时dispatch多个action

我以前的做法是同时dispatch2个action，每个action更新state的不同部分，原则是以数据源区分action。随着深入了解，这是很不科学的，dispatch action引发state更新，state更新引发组件rerender，这个过程是个"类异步"（不是真正的异步）过程，你没办法保证哪个action先更新state触发rerender。有可能一前一后，也有可能，a先要被更新，但是组件的`isBatchingUpdates`为false,处于等待更新状态，这时b的更新需求来了，也等待了一阵直到`isBatchingUpdates`为true，最后2个state部分引发的组件更新被合并更新了，也就是值触发一个rerender。这种不确定性是可怕的，尤其是当2个state之间有依赖或者逻辑关联时，那就会出bug了。如果action中有异步操作，顺序就更不能保证了。

## 通过action.type分发

这种情况比较靠谱的方案是通过相同的action.type执行2部分的state更新。

ListView中：

	function changeAvator(avator,index){
	    let myInfo = Object.assign({},this.props.list[index],{
	        avator:avator
	    });
	    this.props.dispatch(changeUserInfo(userInfo,index));
	}

list页的action：

	// list.js
	export function changeUserInfo(userInfo,index){
	    return {
	        type: CHANGE_MY_INFO,
	        userInfo: userInfo,
	        index:index
	    }
	}

list页的reducer:

	import * as ActionTypes from '../constants/ActionTypes';
	const initialState = {
	    totalPage: 1,
	    pageIndex: 0, 
	    list:[]
	};
	
	export default function handler(state = initialState, action) {
	    let list = state.list.concat();
	    switch (action.type) {
	        case ActionTypes.CHANGE_MY_INFO:
	            list[action.index] = action.userInfo;
	            return Object.assign({},state,{
	                list:list
	            })
	
	        default:
	            return state;
	    }
	}

userInfo的reducer：

	import * as ActionTypes from '../constants/ActionTypes';
	const initialState = {
	    userName:'',
	    userId:0,
	    avator:''
	};
	
	export default function handler(state = initialState, action) {
	    switch (action.type) {
	        case ActionTypes.CHANGE_MY_INFO:
	            return Object.assign({},state,{
	                userName:action.userInfo.userName,
	                userId:action.userInfo.userId,
	                avator:action.userInfo.avator
	            })
	
	        default:
	            return state;
	    }
	}

可以看到通过在不同reducer中对同一action.type处理，更新各自管理的数据，可以达到需求。

以上内容只是我在项目中使用的方式，不一定是最好的，特别是通过action.type分发state的不同部分。如果你有更好的方法，欢迎留言指导。
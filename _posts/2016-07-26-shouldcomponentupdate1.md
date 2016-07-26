---
layout: post
title: "打入react&redux(4) - shouldComponentUpdate初步性能优化"
description: "shouldComponentUpdate初步性能优化"
category: tech
tags: [react,redux]
---
{% include JB/setup %}

# 性能优化的必要性

使用react如果不手动使用shouldComponentUpdate，性能会有很大问题，如果你曾在render函数上打过断点就会知道事情的严重性。

拿官方的一张图来看：

![image](https://echizen.github.com/assets/blog-img/should-component-update.png)

先不管红绿，图注啥的，先假设我们有这种树形组件结构。

任由react本身的话，父组件的重绘一定会导致子组件的重绘，也就是说如果c3有数据更新，无论这个被更新的数据有没有传入到c6,c7,c8，都会导致c6,c7,c8跟着c3发生rerender。这是不合理的低效行为。如果c1更新了，大家就都跟着更新了。。。

再看如果c6组件数据props.c6Data有更新，引起c6、c3、c1重绘不可避免，但是也会引起c7、c8重绘，即使c7、c8没有用到c6组件更新的数据,props上就没有挂载c6Data。这就及其不合理了。

不要惊讶这个现象，如果你不敢相信我的言论，你可以在c7这类子组件的render函数上打断点，然后触发一个只更新c6组件使用的数据的操作，你会看到c7也会再次render的。第一次知道真相的我内心也是崩溃的。。。

所以如果不用shouldComponentUpdate做性能优化，redux中通过combineReducers组合的属于你的根组件上的state中的数据任何一部分有更新，你的根组件及其子组件就跟着一起重绘。。。其实深切的觉得react内部应该做这件事，如果依赖的数据及props上的数据没有更新的，组件就不应该去重绘。不过我觉得没用，我还觉得js应该内置深拷贝方法呢。。。

所以知道真相得我开始性能优化。

# shouldComponentUpdate - 提高性能的救兵

shouldComponentUpdate函数在componentWillUpdate、render函数之前执行，默认返回true，表示组件应该更新，返回false是阻止更新，此时render将不再发生。

因此可以在shouldComponentUpdate中进行判断，只有组件自身关心的数据发生了变化才return true，否则return false.减少重绘次数。

## PureRenderMixin

react提供了`PureRenderMixin`，可以统一帮我们处理`shouldComponentUpdate`的返回值。避免手动判断的繁琐，它会判断`nextprops`和`this.props`以及`nextState`和`this.state`的每个属性是否发生变化，发生变化就`return true`,否则`return false`。

因为不确定它是如何比较的，我特地去看了代码，贴出来分享一下。

ReactComponentWithPureRenderMixin.js：

	'use strict';
	var shallowCompare = require('shallowCompare');
	var ReactComponentWithPureRenderMixin = {
	  shouldComponentUpdate: function(nextProps, nextState) {
	    return shallowCompare(this, nextProps, nextState);
	  },
	};
	
	module.exports = ReactComponentWithPureRenderMixin;
	
shallowCompare.js：

	'use strict';
	var shallowEqual = require('shallowEqual');
	function shallowCompare(instance, nextProps, nextState) {
	  return (
	    !shallowEqual(instance.props, nextProps) ||
	    !shallowEqual(instance.state, nextState)
	  );
	}
	
	module.exports = shallowCompare;

react.js

	function is(x, y) {
	  // SameValue algorithm
	  if (x === y) {
	    // Steps 1-5, 7-10
	    // Steps 6.b-6.e: +0 != -0
	    return x !== 0 || 1 / x === 1 / y;
	  } else {
	    // Step 6.a: NaN == NaN
	    return x !== x && y !== y;
	  }
	}

	function shallowEqual(objA, objB) {
	  if (is(objA, objB)) {
	    return true;
	  }
	
	  if (typeof objA !== 'object' || objA === null || typeof objB !== 'object' || objB === null) {
	    return false;
	  }
	
	  var keysA = Object.keys(objA);
	  var keysB = Object.keys(objB);
	
	  if (keysA.length !== keysB.length) {
	    return false;
	  }
	
	  // Test for A's keys different from B.
	  for (var i = 0; i < keysA.length; i++) {
	    if (!hasOwnProperty.call(objB, keysA[i]) || !is(objA[keysA[i]], objB[keysA[i]])) {
	      return false;
	    }
	  }
	
	  return true;
	}
	
	module.exports = shallowEqual;

从`shallowEqual`函数可见它是先判断objA和objB是否相同，然后通过`is(objA[keysA[i]], objB[keysA[i]]`循环逐个比较2各对象的一级属性是否相等（使用了`Object.keys`）。

is函数只是做简单的`===`判断，又指判断了2个需要比较对象的一级属性（我所说的的一级属性是指：objA.a的a属性，objA.a.b中的b属性即为更深层级的属性）。这个判断是否准确就得看对象是深拷贝还是浅拷贝了。

# 深拷贝和浅拷贝。

### 浅拷贝类似指针

浅拷贝，譬如：

	let obj1 = {
		name:'obj'
	}
	let obj2 = obj1
	let arr1 = [1,2,3]
	let arr2 = arr1;
	
浅拷贝的操作并不会开辟新的内存地址，obj2的地址会指向obj1，只是建立了个类似指针引用的关系。改变obj2会导致obj1的改变

### 深拷贝会分配新的内存地址

	let obj1 = {
		name:'obj'
	}
	let obj2 = Object.assign({},obj1)
	
	let arr1 = [1,2,3]
	let arr2 = arr1.concat();
	
这时，obj2和arr2都已被分配了新的内存地址，并将obj1、arr1的值复制了过去，已经获得独立，改变obj2和arr2并不会影响obj1和arr1的值。

但是注意的是`Object.assign`和`concat`方法只是对当前操作对象进行深拷贝，如果当前操作的是一个复杂对象，并不会自动对他的子属性都做深层拷贝:

![image](https://echizen.github.io/assets/img-blog/QQ20160726-0@2x.png)

这就尴尬了。导致如果我们如果想要做深拷贝就得一层层的手动`Object.assign`，这是非常不方便的，我们需要一个库来提高生产力。

## react.addons.update

**注意这个是低层次的深拷贝**，虽然官方文档把他归为Immutability Helpers，但是他跟Object.assign并没有什么太大区别，都是最要复制的元素深拷贝，但是对元素的复杂类型的子属性还是地址引用。

可以看一下实现react.addons.update的核心方法`shallowCopy`（方法名就叫浅拷贝）：

	function shallowCopy(x) {
	  if (Array.isArray(x)) {
	    return x.concat();
	  } else if (x && typeof x === 'object') {
	    return Object.assign(new x.constructor(), x);
	  } else {
	    return x;
	  }
	}
	var nextValue = shallowCopy(value);

所以不要把react.addons.update当救星，根本没啥卵用，跟自己使用object.assign、concat并无区别，甚至都没有简化代码。只能说如果你不知道object.assign({},value)实现拷贝的话它算不彻底的救了你一下。

# PureRenderMixin的局限

说了这么多，**由于PureRenderMixin的shallowEqual只是简单的检查直接属性是否相等，因此问题的关键是你是否能实现纯粹的深拷贝**。

我们来看看PureRenderMixin能做什么，不能做什么。分3种情况：

## 浅拷贝

	this.state.name='changed'

类似obj2=obj1这种复制，那么PureRenderMixin的shallowEqual函数（浅比较）无疑会得出`nextState.name=this.state.name`，然后return false，那么你将得不到更新！！！所有改动都不会导致重新渲染。

不过我觉得稍微看过react文档的人都知道这种直接改state的方式是不对的，这种错误还是不会犯得。但是冷不防就会出现第二种情况。

## 不完全的深拷贝

整个复杂点的数据作为事例：

	let state = {
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
	};
	
	let firTitle = 'fir-sec';
	
	let newContent = Object.assign({},state.content);
	newContent.section[0].title = firTitle;
	
	let newState = Object.assign({},state,{
		content:newContent
	});
	
	console.log(state.content === newState.content);//true
	console.log(state.content.section === newState.content.section);//false
	console.log(state.content.section[0] === newState.content.section[0]);//false
	
判断是否相等全都用`===`简单处理了，因为react.js里的is方法其实也是这样，只是针对各种情况做了容错。

把这个数据结构和上面的图做个对应:

c1.props = state
c3.props = c1.props.content
c6.props = c3.props.section

`state.content === newState.content`被判为true，导致的严重后果就是PureRenderMixin的shallowEqual函数在这一层上不更新组件，在这个例子里就是本该更新的c3组件不更新了，导致bug。

## 纯粹的深拷贝

	let state = {
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
	};
	
	let firTitle = 'fir-sec';
	
	let newFirSec = Object.assign({},state.content.section[0]);
	let newSection = state.content.section.concat();
	newSection[0] = newFirSec;
	
	let newContent = Object.assign({},state.content,{
		section:newSection
	});
	
	let newState = Object.assign({},state,{
		content:newContent
	});
	
	console.log(state.content === newState.content);//false
	console.log(state.content.section === newState.content.section);//false
	console.log(state.content.section[0] === newState.content.section[0]);//false

	
这样万事皆好，一切正常。

但是这个结果并不能让人开心，因为你得手动这么保证每一层数据都是深拷贝，4次数据就得这样操作4次，太麻烦，即使抽个方法递归调用深拷贝，性能也会不高，而且很麻烦。

# 结论

shouldComponentUpdate用来优化性能，但是，react提供的PureRenderMixin组件判断`nextprops`和`this.props`以及`nextState`和`this.state`是否相等用的是简单的浅层次的比较，只比较直接属性是否`===`，因此需要做到深拷贝。

手动递归利用`Object.assign`和`concat`实现深拷贝既麻烦又性能低下，所以对于数据结构复杂层级深的项目，终极方案是结合`Immutable`使用。

但是，Immutable体积庞大，语法不友好，我在将一个原本未引用改库的项目中改成用这库，改动非常大。Immutable已完成项目的接入成本高，适合一开始就接入，（使用方案我折腾完再分享。。。）

**对于数据结构扁平化，层级浅（3层以下），数据不复杂的项目，做到深拷贝，使用PureRenderMixin就能对性能做好良好的优化，且复杂度和接入成本都很低**

# 黄金外链

[https://facebook.github.io/react/docs/pure-render-mixin.html](https://facebook.github.io/react/docs/pure-render-mixin.html)

[https://facebook.github.io/react/docs/advanced-performance.html](https://facebook.github.io/react/docs/advanced-performance.html)



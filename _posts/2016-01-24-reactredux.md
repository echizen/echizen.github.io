---
layout: post
title: "react+redux防坑技能"
description: "react+redux注意事项"
category: tech
tags: [react,redux]
---
{% include JB/setup %}

1. state不会立即变更

	关于redux下的state变更建议使用react-logger组件。非redux的state变更在debugger时就只能靠console.info了。
	
	但是react 的state并不是在调用this.setState后就立即变更的，react也有类似脏检查的机制（具体是什么机制我还没有研究。。。），会在定时器或者其他条件触发后一起更新state。这时想要打印更新后的state可以使用：
	
		setTimeout(function(){
	    	console.log(this.state.param);
	    }.bind(this),0);
	    
2. 依赖外部状态又自建内部状态的组件一定要使用`componentWillReceiveProps`来获取外部状态变更信息。

	`componentWillReceiveProps`在组件关联的props变化时被调用。
	
		componentWillReceiveProps(nextProps){
	      if(nextProps){
	        this.setState({
	            value: nextProps.value
	        });
	      }
	   }
	   
3. `componentWillReceiveProps`在组件第一次渲染时不会调用
	
	这句话看似不难理解，其实很容易踩坑。第一次并不只是页面加载渲染时。当你新增了某个数据，这个新增的数据被组件渲染时也不会调用该方法。我曾做过个需求是一个table要求tr数据能够新增，并且新增的tr展示为编辑状态。我通过设置组件内部的state.editing标识是否是编辑状态，通过this.data.id是否为空来判断该数据是否为新增的数据。新增的数据为编辑状态应该在constructor中编写，而不是在`componentWillReceiveProps`中。
	
	   constructor(props, context) { 
		    super(props, context);
		    this.state = {
		      editing: this.props.data.id?false:true,
		      data:this.props.data || {}
		    }
	   }
	   componentWillReceiveProps(nextProps){
	      if(nextProps){
	          this.setState({
	              data:nextProps.data
	          });
	      }
	  }
	  
4. 永远不要改变原始的state

	说起来容易，很容易犯错，你需要对数组和对象赋值是浅拷贝有深刻认识，不是`copyObj=obj`，再去改copyObj就能不会影响obj的。
	
	- 对于react管理的state，永远不要在非`construcot`函数中使用`this.state=XXX`这种方式赋值，统一走`this.setState`，这个接口的状态转换才能被react内部各机制接收到变更信息。
		+ 数组：可以采用`slice()`拷贝一个新数组使用。
		+ 对象：`newObj = Object.assign({},this.state.data)`;
		
	- 对于redux管理的state，可以在reducer中处理函数里先建一个nextState，在nextState里操作。`let nextState = Object.assign({},this.state)`，在对nextState进行任意想要的操作。
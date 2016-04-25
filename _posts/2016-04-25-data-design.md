---
layout: post
title: "嵌套数据的扁平化设计"
description: "嵌套数据的扁平化设计，高效快捷操作便携数据"
category: tech
tags: [js]
---
{% include JB/setup %}

前端写了2年，感觉到复杂度来源于数据源，也就是redux中的state。如果数据结构设计的不方便操作，后继的状态改变，交互过程都会变得很复杂。

##扁平化的数据设计

通常我们的数据是嵌套的，按自然关系划分一层层嵌套，数组里含对象，对象里嵌数组或数组。然后查找时一层层的map，效率又低又难懂。

有一种数据设计是扁平化的设计，避免嵌套，只有一层，自然关系用添加一个关联字段表示。这样增查改删都会方便很多。玩过数据库会知道，这类似于关联表的设计。

举个别人家的官方例子：

自然关系表示的：

	articles: [{
	    id: 1,
	    title: 'Some Article',
	    author: {
	      id: 7,
	      name: 'Dan'
	    },
	    ...
	  },
	  ...
	]

扁平化设计：

	articles: {
	  1: { author: 7, ... }, // <--- Same happens for references to other entities in the schema
	  2: { ... },
	  ...
	},
	users: {
	  7: { ... },
	  ..
	}
	
数据结构是想表示一个文章列表的数组对象，每个文章都有对应的作者，作者信息又是个对象，自然关系的表示方法是将作者信息挂在每个文章下面的author字段，形成自然又复杂的嵌套结构，很容易看懂关系，但是不容易操作。

扁平化的设计是讲文章列表和作者列表分开，各自是个对象，每个内容都有唯一id，然后在文章`articles`中用`author`字段的id值表述关联关系。

可能扁平化的结构看上去不是一目了然的知道数据之间的关系（其实习惯了也很自然）），但是操作一下深有感触会方便很多。

## 实例场景-一个多级联动select

下面通过我上周遇到的一个多级联动的select，比较一下2种设计。（为了方便我直接搬运过来了就不抽象了）

开发环境是react+redux，这种select的个数不确定，下一级select由上一级选中的内容确定，且上级select选中后，如果状态里没有它的子select内容，需要去拉取数据填充。所以涉及到的操作有增查。

![image](https://echizen.github.io/assets/blog-img/QQ20160425-0@2x.png)

### 嵌套结构

数据结构，这是一个树状的结构，每个不在末枝的类目都有自己的子支（这里假设他有四层）。

searchPattern中的pageBranch是要搜索的select选中值的数组（业务需要）。deepIndex是被选中的值在所在数组中的索引，是为了查找方便设置的一个变量。

	const initialState = {
	
	    searchPattern:{
	        keyword: "",
	        pageBranch:['']   
	    },
	
	    branches:[{
	        name:'团购',
	        child: [{
	            name:'剁手团',
	            child:[{
	                name:'子分支1'
	            },{
	                name:'子分支2'
	            }]
	        },{
	            name:'大牌'
	        }]
	    },{
	        name:'女装',
	        child: [{
	            name:'春装',
	            child:[{
	                name:'针织衫'
	            },{
	                name:'毛衣'
	            }]
	        },{
	            name:'夏装',
	            child:[{
	                name:'裙子',
	                child:[{
	                	name:'长裙'
	            	},{
	                	name:'短裙'
	            	}]   
	            },{
	                name:'T恤'
	            }]
	        }]
	    }],
	
	    deepIndex:[]    //记录select中分支层级被选中的值的索引
	};
	
（以下部分没注意对象的引用浅拷贝的问题，不小心改变了原状态，由于这部分代码被舍弃了，所以也没为了例子去改，不是重点不要太在意。。。）	
reducer中对state的更新部分：

    case ActionTypes.GET_GROUPS:
       return Object.assign({},state,{
            branches: action.branches
        });

    case ActionTypes.CHANGE_SELECT:
        let newState = Object.assign({},state);
        let deepLen = newState.deepIndex.length;
        if(action.deep != deepLen-1){
            for(let i=deepLen-1;i>action.deep;i--){
                newState.deepIndex.pop();
            }
        }
        newState.deepIndex[action.deep]=action.valueIndex;
        return newState;


View部分，通过循环变量deepIndex来确定展示几个select,通过`tempBranch = tempBranch.child[item]`，让tempBranch这个临时变量再循环中不断的一层层取子支内容展示。还得有个deep变量传给action告诉自己当前操作的是第几层select。

	export default class GroupSelectView extends Component {
	  constructor() {
	    super();
	  }
	
	  handleSelectChange(dataKey,deep,value,valueIndex){
	    this.props.dispatch(changeSelect(deep,valueIndex-1));
	    this.props.dispatch(getGroups(deep));
	
	    let newPattern = Object.assign({},this.props.searchPattern);
	
	    let deepLen = this.props.searchPattern[dataKey].length;
	    if(deep != deepLen-1){
	        for(let i=deepLen-1;i>deep;i--){
	            newPattern[dataKey].pop();
	        }
	    }
	    newPattern[dataKey][deep] = value;
	    this.props.dispatch(changeSearchPattern(newPattern));
	  }
	
	  render() {
	    let tempBranch = this.props.branches.slice();
	    let firOptions = tempBranch.map(function(firBranchItem){
	      return {
	          text:firBranchItem.name,
	          value:firBranchItem.name
	        }
	    });
	    firOptions.unshift({
	      text:'选择业务',
	      value:''
	    });
	
	    let GroupSelect = this.props.deepIndex.map(function(item,index) {
	      if(index==0){
	        tempBranch = tempBranch[item];
	      }else{
	        tempBranch = tempBranch.child[item];
	      }
	      let options = tempBranch.child&&tempBranch.child.map(function(branchItem){
	        return {
	          text:branchItem.name,
	          value:branchItem.name
	        }
	      })
	
	      if(!options){
	        return;
	      }
	
	      options.unshift({
	        text:'选择业务',
	        value:''
	      });
	
	      return (
	          <div className="fl pure-form select-pagebranch" key={index}>
	            <SelectItem label="" value={this.props.searchPattern.pageBranch[index+1]||''} 
	              options={options} onSelectChange={this.handleSelectChange.bind(this,'pageBranch',index+1)}
	            />
	          </div>
	        )
	      
	    }.bind(this))
	
	    return (
	      <div className="fl">
	          <div className="fl pure-form select-pagebranch">
	            <SelectItem label="" value={this.props.searchPattern.pageBranch[0]} 
	              options={firOptions} 
	              onSelectChange={this.handleSelectChange.bind(this,'pageBranch',0)}
	            />
	          </div>
	          {GroupSelect}
	      </div>
	    );
	  }
	}
	
Action中的操作，查找基本靠deepIndex中的索引。这里更变态，由于层级不确定，非常不方便把请求拉取到的select列表内容添加到state中，借用了对象是引用模式，将层级根据deep深度一层层拨开添加到childBranch中，数组childBranch的第一项是总树state.branch，第二项是选中的第一个select内容及其子树，第二项是选中的第二个select内容及其子树。直到当前select的最后一项也就是childBranch的最后一个值，然后获取他的子支内容后添加到他的child属性上，由于对象的赋值是引用复制，改变childBranch的最后一个值的内容也就是改变了第一项的内容。再将新的branch传给reducer更新状态。这变态方法无可避免的改变了原state，因为利用了对象引用赋值的特性。

	//deep:当前操作的select层级
	export function getGroups(deep){
	  return (dispatch, getState) => {
	    let state = getState(),
	        deepIndex = state.deepIndex,
	        childBranch = [state.branches.slice()],
	        resBranch =  Object.assign({},state.branches);
	    
	    // 取根节点
	    if(deep==-1){
	      resBranch = [{
	        name:'团购'
	      },{
	        name:'女装'
	      }];
	    }else{
	      for(var i = 0 ; i <= deep ; i++){
	        if(i==0){
	          childBranch.push(childBranch[i][deepIndex[i]]);
	        }else{
	          childBranch.push(childBranch[i].child[deepIndex[i]]);
	        }
	      }
	
	      if(childBranch[deep+1].child&&childBranch[deep+1].child.length){
	
	      }else{
	        //获取数据data = [{name:'裙子'},{name:'T恤'}]
	        //正式
	        //childBranch[deep+1].child = data;
	      }
	      resBranch = childBranch[0];
	    }
	
	    return dispatch({
	      type:types.GET_GROUPS,
	      branches:resBranch
	    })
	  }
	}
	
	export function changeSelect(deep,valueIndex) {
	  return {
	    type:types.CHANGE_SELECT,
	    deep:deep,
	    valueIndex:valueIndex
	  }
	}


### 扁平结构

数据设计，每个层级都是个单独的对象数组，通过parentId相关联。虽然一眼看上去没有树状结构那么清晰，但是好管理很多。由于branches现在是个对象，所以deepIds存的是键值，这里用了数字键值是为了好累加和操作。

	const initialState = {
	
	    searchPattern:{
	        keyword: "",
	        group:[''] 
	    },
	    branches:{
	        0:[{
	            id:'001',
	            name:'团购'
	        },{
	            id:'002',
	            name:'女装'
	        }],
	        1:[{
	            id:'011',
	            parentId:'001',
	            name:'剁手团'
	        },{
	            id:'012',
	            parentId:'001',
	            name:'大牌'
	        },{
	            id:'013',
	            parentId:'002',
	            name:'春装'
	        },{
	            id:'014',
	            parentId:'002',
	            name:'夏装'
	        }],
	        3:[{
	            id:'101',
	            parentId:'011',
	            name:'子分支1'
	        },{
	            id:'102',
	            parentId:'011',
	            name:'子分支2'
	        },{
	            id:'101',
	            parentId:'013',
	            name:'针织衫'
	        },{
	            id:'102',
	            parentId:'013',
	            name:'毛衣'
	        },{
	            id:'103',
	            parentId:'014',
	            name:'裙子',
	        },{
	            id:'104',
	            parentId:'014',
	            name:'T恤',
	        }]
	    },
	
	    deepIds:['']   //记录当前选中了的树枝层级的id
	};
	
reducer中state转换：

 	case ActionTypes.GET_GROUPS:
        return Object.assign({},state,{
            branches: action.branches
        });

    case ActionTypes.CHANGE_SELECT:
        let newDeepIds = state.deepIds.concat();
        let deepLen = newDeepIds.length;
        if(action.deep != deepLen-1){
            for(let i=deepLen-1;i>action.deep;i--){
                newDeepIds.pop();
            }
        }
        newDeepIds[action.deep] = action.id;
        
        return Object.assign({},state,{
            deepIds:newDeepIds
        });


View中，依旧是遍历deepIds，通过parentId查找branches[deep+1]中是当前select选中状态内容的子支，保存到tempChildBranch中，再渲染页面。

	export default class GroupSelectView extends Component {
	  constructor() {
	    super();
	  }
	 
	  handleSelectChange(deep,oneBranch,value,index){
	    var id = oneBranch[index-1].id;//第一个为提示语
	    this.props.dispatch(changeSelect(deep,id));
	    this.props.dispatch(getGroups(deep,id));
	
	    let newBranch = this.props.searchPattern.group.slice();
	
	    let deepLen = newBranch.length;
	    if(deep != deepLen-1){
	        for(let i=deepLen-1;i>deep;i--){
	            newBranch.pop();
	        }
	    }
	    newBranch[deep] = value;
	    let newPattern = Object.assign({},this.props.searchPattern,{
	      group:newBranch
	    })
	    this.props.dispatch(changeSearchPattern(newPattern));
	  }
	
	  render() {
	    let tempBranch = Object.assign({},this.props.branches);
	    let tempChildBranch = [];
	    let firOptions = tempBranch[0].map(function(firBranchItem){
	      return {
	          text:firBranchItem.name,
	          value:firBranchItem.name
	        }
	    });
	    firOptions.unshift({
	      text:'选择业务',
	      value:''
	    });
	
	    let GroupSelect = this.props.deepIds.map(function(parentId,index){
	      
	      tempChildBranch = tempBranch[index+1] && tempBranch[index+1].slice().filter(function(item){
	        return item.parentId == parentId;
	      })
	    
	      if(tempChildBranch&&tempChildBranch.length==0){
	        return;
	      }
	
	      let options = tempChildBranch&&tempChildBranch.length&&tempChildBranch.map(function(branchItem){
	        return {
	          text:branchItem.name,
	          value:branchItem.name
	        }
	      })
	
	      if(!options){
	        return;
	      }
	
	      options.unshift({
	        text:'选择业务',
	        value:''
	      });
	
	      return (
	          <div className="fl pure-form select-branch" key={index}>
	            <SelectItem label="" value={this.props.searchPattern.group[index+1]||''} 
	              options={options} 
	              onSelectChange={this.handleSelectChange.bind(this,index+1,tempChildBranch)}
	            />
	          </div>
	        )
	    }.bind(this))
	
	    return (
	      <div className="fl group-select-wrap">
	           <div className="fl pure-form select-branch">
	            <SelectItem label="" value={this.props.searchPattern.group[0]||''} 
	              options={firOptions} 
	              onSelectChange={this.handleSelectChange.bind(this,0,tempBranch[0])}
	            />
	          </div>
	
	          {GroupSelect}
	      </div>
	    );
	  }
	}
	
Action,由于每个层级的树枝select内容都是一个单独的对象数组，且branches是数字键值，通过代表层级深度的deep很容易获得到下一级树枝state.branches[deep+1]，需要获取的数据也是往这一层的对象数组里添加，注意的就是用深拷贝方式不要修改原数组。如果有需要更新，更新的都是下一层级`branches[deep+1]`的内容。

	export function getGroups(deep,parentId){
	  return (dispatch, getState) => {
	    let state = getState(),
	        resBranch =  {},
	        childBranch = state.branches[deep+1],
	        newBranchChild = [],
	        isNew = true,
	        testDate = [];
	     
	    // 取根节点
	    if(parentId==null){
	    	//根节点特殊处理，测试数据
	      resBranch = {
	        0:[{
	            id:'001',
	            name:'团购'
	        },{
	            id:'002',
	            name:'女装'
	        }]
	      };
	    }else{
	      childBranch&&childBranch.map(function(item){
	        if(item.parentId==parentId){
	          isNew = false;
	        }
	      });
	
	      if(isNew){
	        // get data from server
	        //获取数据testData = [{
	                id:'013',
	                parentId:'002',
	                name:'春装'
	            },{
	                id:'014',
	                parentId:'002',
	                name:'夏装'
	            }];
	                   
	        if(childBranch&&childBranch.length){
	          newBranchChild = [...childBranch,
	                            ...testDate
	                          ];
	        }else{
	          newBranchChild = testDate;
	        }
	        
	        resBranch = Object.assign({},state.branches,{
	          [deep+1]:newBranchChild
	        })
	      }else{
	        return;
	      }
	    }
	
	    return dispatch({
	      type:types.GET_GROUPS,
	      branches:resBranch
	    })
	  }
	}
	
	export function changeSelect(deep,id) {
	  return {
	    type:types.CHANGE_SELECT,
	    deep:deep,
	    id:id
	  }
	}


### 体验

之前的嵌套形似为了完成这个需求用了变态的对象引用一改都改的奇技淫巧，并不是十分的科学，我没有想到科学的方式。使用扁平化设计后操作变得简单清晰多了，而且扁平化设计完成功能的时间只有嵌套方式的1/5，不到一小时。嵌套太绕了，容易把自己弄晕。


## 安利

虽然扁平化方式的数据设计看上去数据关系没有那么清晰，但是相信我，如果你的数据关系够复杂，嵌套层级够深，扁平化设计绝对让你在增查改删的操作上大占优势。

然后推荐个repo工具，我第一次看到扁平化数据设计在前端的运用是通过这个repo的，虽然看了一眼觉得其实用的就是关联数据表的思想，而且自己设计的好，完全可以抛开这个库自己实现，但是对于处理自己没办法掌握的接口数据时确实这个库能带来便利。

[normalizr](https://github.com/gaearon/normalizr)
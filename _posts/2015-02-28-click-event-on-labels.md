---
layout: post
title: "click事件在label标签的checkbox、radio中执行2次"
description: "click event on labels，关注元素的默认事件，默认事件和绑定事件重合仍会2个都执行。angularJs ng-click在label上执行2次。阻止事件冒泡"
category: tech
tags: [js,html,angularJs,前端,pro]
---
{% include JB/setup %}

这是2个月前遇见的事，前一阵有人问我，便想着要记下来。以下叙述不仅针对于angularJs的ng-click事件，对普通的click事件也适用。

*示例一*：

	<label class="mul_col check_area" ng-click="checkGoods($index)">
	    <span class="wait_check"  ng-class="{'checked': item.isChecked}" ></span>
	    <input type="checkbox" name="isChosed[]" class="isChose_checkbox" value="{{item.cartId}},{{item.nub}}" />
	</label>
	
js中是这么写的：

	$scope.checkGoods = function(index){
	    $scope.items[index].isChecked = !$scope.items[index].isChecked;
	}
	
视觉感受是click事件无效了，因为`.wait_check`的class没有变化，`item.ischecked`的值也没有变化。分布调试后，发现其实是执行了2次checkGoods()，也就是`ng-click`事件被执行了2次。

## 元素默认绑定click事件

一些元素如`<a>、<button>、<input type="checkbox">`本生就默认绑定了click事件，即使你不绑定，click事件发生时他们也会接收到。

当label标签包含input时，label也自动监听原来input默认监听的事件。

## 事件冒泡

我们知道事件传递方式有2种：事件捕获和事件冒泡。事件会从父元素传到子元素，再从子元素传到父元素，如果事件绑定发生在父元素传到子元素的过程中，则称为事件捕获传递，如果事件绑定发生在子元素传到父元素的阶段，则称为事件冒泡传递。

	<div id="parent">
	    <div id="child"></div>
	</div>

由于父元素包含子元素，子元素属于父元素，所以在子元素上的操作就相当于在父元素上的操作，譬如`#parent`绑定了click事件，则点击子元素`#child`，会自然的触发原本绑在`#parent`上的click事件。准确的说这其实是子元素继承了父元素的事件，使子元素父元素绑定了相同的事件。

什么时候会发生事件传递呢？**父元素和子元素必须绑定了相同的事件**。我们会觉得子元素继承父元素的事件很正常，这是浏览器规则帮我们做的，但往往忽视自己绑定的事件，如果我们这么做：

    $('#child').click(function(){
        callChildEvt();
    })

    $('#parent').click(function(){
        callParentEvt();
    })
    
当我们点击了`#child`后，会发现`callChildEvt()`先执行，随后`callParentEvt()`也执行了。其实这就是事件冒泡，传递给了`#parent`。

## lable control

label标签的功能不用多说，它把所包含的input的用户交互区域扩展了.**注意的是label和所包含的input都开始绑定默认事件，此时会发生事件冒泡现象**。

在label上的click事件的处理函数会触发2次就是由于：第一次是label自己接收到事件，执行处理函数，第二次是input接受到事件后冒泡传递给label，再次触发处理函数。

再看一例：

*示例二*：

	<html>
	    <head>
	        <meta charset="utf-8">
	        <script type="text/javascript" src="jquery-2.0.3.min.js"></script>
	    </head>
	    <body>
	        <label>标题：
	            <input type="checkbox" >
	        </label>
	
	        <script type="text/javascript">
	            $('label').click(function(){
	                console.log('label_click');
	            })
	
	            $('input').click(function(){
	                console.log('input_click');
	            })
	        </script>
	    </body>
	</html>
	
点击label区域运行结果：（不要直接点击input元素）

![image](https://echizen.github.io/assets/blog-img/QQ20150301-1.png)

点击label后，label本身绑定的click事件执行一次，同时input本身也绑定有click事件，冒泡传递给了label，在出发一次函数。

点击input区域：

![image](https://echizen.github.io/assets/blog-img/QQ20150301-2.png)

如果直接点击input区域，会将click事件冒泡传递给label，执行一次label的处理函数。

----

稍作修改，阻止input的冒泡事件：

	$('label').click(function(){
	    console.log('label_click');
	})
	
	$('input').click(function(e){
	    console.log('input_click');
	    e.stopPropagation();
	})
	
再点击label区域运行结果：（不要直接点击input元素）
	
![image](https://echizen.github.io/assets/blog-img/QQ20150301-4.png)

可见掐断事件通过input冒泡传递后，label便不能再接收第二次由input传递过来的事件。

点击input区域：

![image](https://echizen.github.io/assets/blog-img/QQ20150301-3.png)


**注意的是，无论我们有没有人为的给label绑定click事件，其实你点击label后，其上的click事件处理函数都会发生2遍。我们绑定的处理函数和浏览器默认绑定事件的处理函数是同时进行的。所以label被触发的2次click事件并不是由于默认的和我们绑定的**

## 关于本例ng-click的分析

在本例中，点击label区域后，click事件执行一次，触发一次checkGoods()，由于checkbox本身默认也有click事件监听器，所以checkbox又触发一次click，并且将事件冒泡传递给了label，checkGoods()再次被触发。

## 解决方案。

### 在label内部元素上绑定ng-click

将ng-click绑定到input上，便只会触发一次。

	<label class="mul_col check_area">
	    <span class="wait_check"  ng-class="{'checked': item.isChecked}" ></span>
	    <input type="checkbox" name="isChosed[]" class="isChose_checkbox" value="{{item.cartId}},{{item.nub}}" ng-click="checkGoods($index)" />
	</label>
	
### 阻止事件冒泡。
虽然不符合我示例一的功能要求，但是如果你仅仅是想阻止label执行2遍事件，可以在子元素input上阻止事件冒泡，` e.stopPropagation()`,很多情况下避免冒泡都是这样做。但是对于label和input组合这么做伤害了初衷，这样点击input后，对于label来说是无效的，点击input不会触发label的click事件了，不符合常识，有一种子元素在该事件上不属于父元素的感觉，这个是要注意的。


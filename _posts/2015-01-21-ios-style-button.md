---
layout: post
title: "ios 风格checkbox/radio"
description: "ios7/ios8 风格checkbox/radio,仿苹果按钮"
category: tech 
tags: [css3,基础]
---
{% include JB/setup %}

再做web app或hybird app时，为了达到一致的风格，常需要模仿原生的样式，比如说ios的checkbox样式。

![image](https://echizen.github.io/assets/blog-img/QQ20150121-1.png)

简单的内容，直接上代码

### CSS

	.checkbox {
	    height: 28px;
	    width: 50px;
	    position: absolute;
	    top: -3px;
	    right: 6px;
	    z-index: 9997;
	    -moz-border-radius: 35px;
	    -webkit-border-radius: 35px;
	    border-radius: 35px;
	    /*background-color: #efefef;*/
	}
	.checkbox:after {
	    height: 28px;
	    width: 25px;
	    position: absolute;
	    top: 0;
	    left: 0;
	    content: '';
	    z-index: 9996;
	    -moz-border-radius: 35px 0 0 35px;
	    -webkit-border-radius: 35px;
	    border-radius: 35px 0 0 35px;
	    /*background-color: #00CC00;*/
	}
	
	.control {
	    height: 25px;
	    width: 25px;
	    outline: 0;
	    position: absolute;
	    top: 8px;
	    right: 7px;
	    z-index: 9998;
	    -webkit-appearance: none;
	    -moz-border-radius: 36.5px;
	     -webkit-border-radius: 36.5px; 
	    border-radius: 36.5px;
	    background-color: #fff;
	    border: 0;
	}
	.control:checked {
	  right: 30px;
	}
	
	.control + .checkbox{
	    background-color: #ccc;
	}
	.control + .checkbox:after{
	    background-color: #ccc;
	}
	
	.control:checked + .checkbox{
	    background-color: #00CC00;
	}
	.control:checked + .checkbox:after{
	    background-color: #00CC00;
	}
	
	.control:after {
	  height: 40px;
	  width: 40px;
	  position: absolute;
	  right: 15px;
	  top: 15px;
	  /*content: '';*/
	  z-index: 9998;
	  -moz-border-radius: 50%;
	  -webkit-border-radius: 50%;
	  border-radius: 50%;
	  background-color: #c2c0be;
	  *zoom: 1;
	  background-image: -moz-linear-gradient(top, #c2c0be 0%, #d7d7d7 72%);
	  background-image: -webkit-linear-gradient(top, #c2c0be 0%, #d7d7d7 72%);
	  background-image: linear-gradient(to bottom, #c2c0be 0%, #d7d7d7 72%);
	}

### html

	<section class="col-xs-12 top_line">
	    <div class="mul_col title_icon">
	        <img src="image/test.png">
	    </div>
	    <div class="mul_col blue">会议是否公开</div>
	    <div class="chose_button" style="float:right">
	        <input type="checkbox" name="isPublic" id="isPublic" value="0" style="display:none" checked> 
	        <div class="full_col switch">
	            <input type="checkbox" id="control" class="control" name="isPublic" id="isPublic" value="1" <%=meetDetail.get('isPublic')?meetDetail.get('isPublic')==1?'checked':'':''%> >
	            <label for="control" class="checkbox"></label>
	        </div>
	    </div>
	</section>
	
### 分析
没有用js,全是css `:checked`选择器的功劳。
将input的checkbox框设置成圆，在利用label扩大其作用的点击范围（话说label真是一个对表单项有特殊要求样式的一个很实用的标签），将label的样式设置成椭圆。为`:checked`样式设置不同的背景色。

此方法虽简单，但值得一记，用这种思路可以做出很多样式。有时候善于用css,真的可以避免很多的臃肿的js代码。

`:checked`是`pseudo-classes`的用法，类似的`pseudo-classes`还有很多，详见：

[https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes)

---
layout: post
title: "谈谈angularjs directive中的scope"
description: ""
category: tech
tags: [angularjs]
---
{% include JB/setup %}

初创公司忙的没日没夜没周末，开发进度甩博客更新几条街。在印象笔记中这是我三周前准备要写的东西了。。。（打算8月份停一个月，留点时间给自己整理总结）。

directive号称是angularjs最复杂的部分。第一个指令我写了一天，一天的时间才稍微明白了一点指令的使用。学会指令第一步就是要搞懂它的作用域。

指令中template参数是一段有html意义的字符串，或是一个html片段的文件的路径。这段内容会被插入到页面dom中，由angularjs编译渲染。这段html片段与其他的片段无异，也可以进行数据绑定。这时候这段html中的数据来源于哪里就是个问题了。

先上鄙人人生的第一个directive实例：一个手机端的点击发送验证码并进入重发倒计时状态的按钮。

	var smsBtn = angular.module('smsBtnDirective', []);
	
	smsBtn.directive('getCodeBtn',['$interval',function($interval) {
	
	  return {
	    restrict: 'E',
	    replace: true,
	    scope: {
	      'getCodeFun': '&'
	    },
	    template: '<button class="button button-small button-energized wide_small_button flex_center" ng-click="getCode()" style="min-width:92px;">{{timeTip}}</button>',
	    link: function (scope,ele) {
	      var timeFlag = 60;
	      scope.timeTip = '获取验证码';
	
	      var minuteClock = function(){
	          // 验证码重新发送倒计时
	          this.clock = $interval(function(){
	              if(timeFlag > 0 && timeFlag <= 60){
	                  timeFlag--;
	                  scope.timeTip = timeFlag+' s';
	                  
	              }else{
	                  $interval.cancel(this.clock);
	                  timeFlag = 60;
	                  scope.timeTip = '重新发送';
	              }
	          },1000);
	      };
	
	      scope.getCode = function(){
	        var canSend =scope.getCodeFun();
	        if(canSend){
	          minuteClock();
	        }
	      };
	
	      ele.on('$destroy', function() {
	        $interval.cancel(this.clock);
	      });
	
	      
	    }
	  };
	}]);

调用方式：

	<get-code-btn get-code-fun="signupStep1.getCode()"></get-code-btn>
	
相关的调用html片段及js片段

	<div class="item item-input item-button-right">
        <input type="tel" name="smsCode" placeholder="请输入验证码" ng-model="signupStep1.phone.smsCode" required>
        <get-code-btn get-code-fun="signupStep1.getCode()"></get-code-btn>
    </div>
    
controller:

	use strict';
  	var userFindPd = angular.module('jbj.findPassword',['userAuthService','smsBtnDirective']);

  	userFindPd.controller('findPdStep1Ctrl',['findPdSms',function(findPdSms){

    var vm = this;

    vm.getCode = function(){

      if(vm.form.mobilePhoneNumber.$error.phone){
        feedbackMsg.phoneNumberError();
        return false;
      }else{
        findPdSms.get({'mobilePhoneNumber':vm.phone.mobilePhoneNumber},function(data){
          if(data.code===0){
            feedbackMsg.smsCodeSuccess();
          }else{
            feedbackMsg.smsCodeError(data.message);
          }    
        });
        return true;
      }
    };
    
  }]);



## 当前控制器的scope作用域

	scope:false(default)

此时，template直接访问到其所插入的控制器中的scope，可以与当前scope下的数据直接绑定。

## Isolating the Scope ，创建指令独立的作用域。

此时我们需要给scope参数赋予新的对象。

	scope: {
		newmodel: '=oldmodel',
		newstring: '@oldstring',
		newmethod: '&oldmethod'
	}

### ‘=’Two-way model binding(model)

将父scope中的数据与指令scope的变量双向绑定。也就是说oldmodel会和newmodel同步变化。

可能有些疑惑，这和直接使用父作用域而不创建独立作用域有什么区别？为何还要这番折腾。

这是为了组件的复用，当你需要在directive里的link或是compile甚至controller函数中对数据进行预处理，在绑到template中展示时就有意义了。因为你不可能要求每一个复用你的组建的controller都使用相同的变量名。而是创建指令自己的作用域和变量名比较科学。这只是我发觉的一个应用场景。

### ‘@’Attribute string binding(string)

用于在子作用域中获取父作用域中的变量。可以获取{{value}}形式的变量值，获取到的是被渲染后的变量值，但是并不会引起变量的双向绑定。

### '&'Callback method binding(method)

用于在指令作用域中获取父作用域中的方法。

这是很常见的做法，比如回调函数。比如我的示例中的发送验证码的函数，并不能写在directive的link中，因为每一个调用组件的地方获取验证码的接口和处理逻辑都可能不同。为了通用，应该将获取验证码的函数从父scope中引进到子scope中。

在以上3种方式创建了指令自己独立的作用域后。就可以在指令自己的link、compile、controller函数中通过newmodel、newstring、newmethod的变量名对相关变量进行操作了。

另外我建议都使用下面这种简写方式，否则变量太多容易头晕。

	scope: {
		isolatemodel: '=',
		isolatestring: '@',
		isolatemethod: '&'
	}


##黄金外链
[https://docs.angularjs.org/guide/directive](https://docs.angularjs.org/guide/directive)

[http://weblogs.asp.net/dwahlin/creating-custom-angularjs-directives-part-3-isolate-scope-and-function-parameters](http://weblogs.asp.net/dwahlin/creating-custom-angularjs-directives-part-3-isolate-scope-and-function-parameters)

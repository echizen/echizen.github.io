---
layout: post
title: "异步处理-强大的promise"
description: "javascript中的promise"
category: tech
tags: [js, pro]
---
{% include JB/setup %}

使用了angular之后，经常会遇到异步带来的问题，譬如directive要等到controller取到服务器数据后再渲染什么的。逼得我不得不去了解promise.（话说之前还一直以为他是node的一个API，汗颜。。。）

promise已被纳入ES6中，不过浏览器对其的支持性还不是特别好。

![image](https://echizen.github.io/assets/blog-img/QQ20150808-1@2x.png)

硬伤是安卓4.4.4之后才支持，这显然不能商用，还得借住库。

jquery、commonjs、dojo、Q、when 都实现了自己的promise部分。名称略有不同，有叫promise的，有叫q的，有叫deffer的。但大体用法是一样的。他们都遵从了同一套规范 `Promises/A+`，但是jquery有点特殊。

##应用方式

先来假设一个应用场景：函数b需要在函数a执行完后执行。而函数a中有异步请求，譬如ajax,我们需要异步请求完成后根据异步请求成功或者失败来确定函数b执行内容。

那么我们需要在a函数中设置个promise对象，他相当于一个触发器，在你将要运行ajax之前，先设个promise标志，说明接下来要干异步的事了。当a函数中异步操作进行完后，立即通知b函数你可以干你该干的事了。

a函数：

	function a(){
	  var promise = new Promise( function (resolve, reject) {
	  // 在这里做你想做的事，譬如异步请求ajax。
	  // 成功时（譬如ajax请求返回数据）：
	  resolve(value);
	  // 失败时（譬如ajax请求返回错误码）：
	  reject(reason);
	  return promise;
	}

b函数：

	function b(){
	  //...初始化信息
	  a.then(function(value){
	    // 成功时的操作，value是你传入的值
	  }, function(reason){
	    //失败时的操作
	  })
	}
	
##angularjs中的promise

angularjs 中的promise使用的是$q服务，只是换了个名字，本质还是一样的。特别的是他还有notify方法，用来在异步运行途中，不停的告诉对方我还在运行中。

###使用方法

a函数：

	function a(){
	  var deferred = $q.defer();
	  // 在这里做你想做的事，譬如异步请求ajax。
	  // 成功时（譬如ajax请求返回数据）：
	  if(success){
	    deferred.resolve(value);
	  }else{
	    // 失败时（譬如ajax请求返回错误码）：
	    deferred.reject(reason);
	  }
	  return deferred.promise;
	}

b函数：

	function b(){
	  //...初始化信息
	  promise = a();
	  promise.then(function(value){
	    // 成功时的操作
	  },function(reason){
	    //失败时的操作
	  },function(update){
	    //进行中的提示
	  })
	}

angular对promise的调用方式相对更清晰。

###我的运用

测试时测出了我之前写的发送验证码的bug，就是验证码请求失败，提示错误时，我依然在倒计时。。。。然后我将返回值移至异步回调中，然后就永远不会倒计时了。。。

以前的代码是这样写的：

directive中：

	scope.getCode = function(){
      var canSend =scope.getCodeFun();
      if(canSend){
        minuteClock();
      }
    };

controller中：

	vm.getCode = function(){
      if(vm.form.mobilePhoneNumber.$error.phone){
        feedbackMsg.phoneNumberError();
        return false;
      }else{
        getSmsCode.get({'mobilePhoneNumber':vm.phone.mobilePhoneNumber},function(data){
          if(data.code===0){
            if(window.localStorage && window.localStorage.getItem){
              localStorage.setItem('userPhone',vm.phone.mobilePhoneNumber);
            }
            feedbackMsg.smsCodeSuccess();
            return true;
          }else{
            feedbackMsg.smsCodeError(data.message);
            return false;
          }    
        }); 
      }
      
    };
    
很显然由于$resource的使用，是的异步部分的`return true`和`return false`执行有延时，函数如果通过了phone的验证，立即返回的是undefined,undefined在if中被转换成了布尔值false。

使用了promise之后，一切都变得很精确。

directive中：

      scope.getCode = function(){
        var promise = scope.getCodeFun();
        promise.then(function(value) {
          scope.notSubmit = true;
          minuteClock();
        });
      };

controller中：

    vm.getCode = function(){
      var deferred = $q.defer();
      if(vm.form.mobilePhoneNumber.$error.phone){
        feedbackMsg.phoneNumberError();
        deferred.reject('error');
      }else{
        getSmsCode.get({'mobilePhoneNumber':vm.phone.mobilePhoneNumber},function(data){
          if(data.code===0){
            if(window.localStorage && window.localStorage.getItem){
              localStorage.setItem('userPhone',vm.phone.mobilePhoneNumber);
            }
            feedbackMsg.smsCodeSuccess();
            deferred.resolve('success');
          }else{
            feedbackMsg.smsCodeError(data.message);
            deferred.reject('error');
          }    
        }); 
      }
      return deferred.promise;
    };
    
##promise的优点

如果你认真看，会发现promise能实现的回调函数都能实现，所以为什么还要引入promise?

+ promise更清晰，回调函数容易陷入回调黑洞

+ promise更适合错误处理，他专门提供了成功时的接口，失败时的接口。回调的话，你只能传入2个回调函数参数了

+ promise解耦关系变得可能。这是我最大的感悟。如果用回调，上面的示例，是需要在a函数中植入成功或失败的回调函数，代码变得臃肿，而且维护时你必须对照着b函数看a函数中执行的回调函数是什么用处。而promise给出了通用的接口resolve和reject，a、b2个函数可以真正的分开，也方便a函数的复用。而且你无法想象在directive和controller这2个解耦的组件中（连通信都借用scope）使用回调这种紧密关联的东西是一件多么糟糕的事情。

##示例代码（来自MDN）

	'use strict';
	
	// A-> $http function is implemented in order to follow the standard Adapter pattern
	function $http(url){
	 
	  // A small example of object
	  var core = {
	
	    // Method that performs the ajax request
	    ajax : function (method, url, args) {
	
	      // Creating a promise
	      var promise = new Promise( function (resolve, reject) {
	
	        // Instantiates the XMLHttpRequest
	        var client = new XMLHttpRequest();
	        var uri = url;
	
	        if (args && (method === 'POST' || method === 'PUT')) {
	          uri += '?';
	          var argcount = 0;
	          for (var key in args) {
	            if (args.hasOwnProperty(key)) {
	              if (argcount++) {
	                uri += '&';
	              }
	              uri += encodeURIComponent(key) + '=' + encodeURIComponent(args[key]);
	            }
	          }
	        }
	
	        client.open(method, uri);
	        client.send();
	
	        client.onload = function () {
	          if (this.status == 200) {
	            // Performs the function "resolve" when this.status is equal to 200
	            resolve(this.response);
	          } else {
	            // Performs the function "reject" when this.status is different than 200
	            reject(this.statusText);
	          }
	        };
	        client.onerror = function () {
	          reject(this.statusText);
	        };
	      });
	
	      // Return the promise
	      return promise;
	    }
	  };
	
	  // Adapter pattern
	  return {
	    'get' : function(args) {
	      return core.ajax('GET', url, args);
	    },
	    'post' : function(args) {
	      return core.ajax('POST', url, args);
	    },
	    'put' : function(args) {
	      return core.ajax('PUT', url, args);
	    },
	    'delete' : function(args) {
	      return core.ajax('DELETE', url, args);
	    }
	  };
	};
	// End A
	
	// B-> Here you define its functions and its payload
	var mdnAPI = 'https://developer.mozilla.org/en-US/search.json';
	var payload = {
	  'topic' : 'js',
	  'q'     : 'Promise'
	};
	
	var callback = {
	  success : function(data){
	     console.log(1, 'success', JSON.parse(data));
	  },
	  error : function(data){
	     console.log(2, 'error', JSON.parse(data));
	  }
	};
	// End B
	
	// Executes the method call 
	$http(mdnAPI) 
	  .get(payload) 
	  .then(callback.success) 
	  .catch(callback.error);
	
	// Executes the method call but an alternative way (1) to handle Promise Reject case 
	$http(mdnAPI) 
	  .get(payload) 
	  .then(callback.success, callback.error);
	
	// Executes the method call but an alternative way (2) to handle Promise Reject case 
	$http(mdnAPI) 
	  .get(payload) 
	  .then(callback.success)
	  .then(undefined, callback.error);
	  
##黄金外链

[https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)

[http://www.oschina.net/translate/whats-so-great-about-javascript-promises](http://www.oschina.net/translate/whats-so-great-about-javascript-promises)

[https://docs.angularjs.org/api/ng/service/$q](https://docs.angularjs.org/api/ng/service/$q)

[http://www.html5rocks.com/en/tutorials/es6/promises/](http://www.html5rocks.com/en/tutorials/es6/promises/)关于链式调用promise、错误处理catch、forEach处理异步内容部分很精彩
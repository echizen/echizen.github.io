---
layout: post
title: "angularjs directives"
description: "directive使用场景，使用实例"
category: tech
tags: [angularjs]
---
{% include JB/setup %}

指令应用场景很广，多是与dom和js都有关联的部分。

在没有接触到angular之前并不懂得什么是真正的通用，顶多是把可复用的东西抽成函数，放到一个单独的文件里，在再需要的地方引入。但是angular的模块封装、依赖载入的方式让复用变得真正可行。不同项目中的组件都可以复用，将directive文件引入，再在需要调用的模块中依赖复用组建的模块即可。

##通用组件

譬如手机验证码发送的按钮，需要在注册和找回密码时都用到。它有获取验证码的功能，并拥有重新获取验证码倒计时的UI效果。这个另外一篇博客详述了，就不再赘述了。

	作为一个程序员，能偷懒就偷懒，能写一遍复用的东西绝不写两遍。而且要做到通用，就得独立，尽可能不对调用环境产生依赖，考虑到调用环境的多样性。

##jquery插件的封装。

我相信很多初学者在同时使用jquery和angularjs时都是非常的纳闷，我到底该把jquery插件的初始化代码写在哪里。。。

由于jquery很长一段时间的统治地位，和jquery对ui效果dom操作的天然优势，进行一个大项目时完全不依赖jquey插件是很难办到的，而jquery是全局的，$污染了全局作用域，angular在每个controller下都是一个独立的作用域，然后angular和jquery的渲染时间不一样，jquery和原生js（都写在全局时，而不是在angular的模块中）的执行时间先于angular渲染时间。最友好的jquery插件的使用方式，就是封装成directive，在angular的控制下触发。

最主要的是在link函数下使用：

		$(element).pluginName(configData);

这段代码放在link中，当angular解析到页面的指令的element标识符时，开始获取template,并调用link方法链接，此时调用与这段dom相关的插件的初始化操作最合适不过了。

譬如我一直挚爱的特别好用的图片上传裁剪插件[cropit](http://scottcheng.github.io/cropit/)

	var plugin = angular.module('jqueryPluginDirective', []);
	
	// 图片上传裁剪处理插件
	plugin.directive('uploadCropImg',function() {
	
	  return {
	    restrict: 'E',
	    replace: true,
	    scope: {
	      cropitConfig: "=",
	      renderTime : "=",
	      uploadImgdata : "&"
	    },
	    template: '<div class="image-cropper clear-padding" id="coverImg" style="margin-left:24px;">'+
	                  '<div class="cropit-image-preview-container" >'+
	                      '<div class="cropit-image-preview" style="width: 540px;height: 300px;"></div>'+
	                  '</div>'+
	                    
	                  '<div class="slider-wrapper" style="min-height: 30px;padding-top: 10px;">'+
	                      '<span class="glyphicon glyphicon-picture" style="font-size:14px;"></span>'+
	                      '<input type="range" class="cropit-image-zoom-input custom" min="0" max="1" step="0.01">'+
	                      '<span class="glyphicon glyphicon-picture" style="font-size:18px;"></span>'+
	                  '</div>'+
	                    
	                  '<input type="file" class="cropit-image-input" id="trueChoseImg"/>'+
	
	                  '<button class="btn btn-primary select-image-btn" type="button" ng-click="choseImg()"><span class="glyphicon glyphicon-picture"></span>选择图片</button>'+
	
	                  '<button class="btn btn-success js_upload_img" type="button" ng-click="uploadImg()"><span class="glyphicon glyphicon-cloud" ></span>上传图片</button>'+
	                '</div>',
	    link: function (scope, element) {
	      $(element).cropit(scope.cropitConfig);
	                                                         
	      scope.choseImg = function(){
	          $('#trueChoseImg').trigger('click');
	       };
	
	      scope.uploadImg = function(){
	            //要传输的图片数据
	            var imgData = {
	              'filePath':$('.cropit-image-input').val(),
	              'base64ImgData':$('.image-cropper').cropit('export')
	            };
	            // 执行传输图片数据的函数（由外部controller定义的函数）
	            scope.uploadImgdata()(imgData);
	
	        // }
	      };
	
	    }
	  };
	
	});
	
噢，原谅我懒惰，这么一大段template没有抽出为文件。

##自定义表单验证规则

除了angularjs内置的几种表单验证规则：required, number, pattern, minlength, maxlength, min, max,开发过程中还需要大量的表单验证规则，写入pattern的正则虽然方便，但不能复用了。所以还是directive最合适。譬如这段验证手机号码的正则。

	var phoneNumberReg = /^(1[3458]\d{9})$|(17\d{9})$/;
	formValidation.directive('phone', function() {
	  return {
	    require: 'ngModel',
	    link: function(scope, elm, attrs, ctrl) {
	      ctrl.$validators.phone = function(modelValue, viewValue) {
	        if (ctrl.$isEmpty(modelValue)) {
	          return false;
	        }
	        if (phoneNumberReg.test(viewValue)) {
	          return true;
	        }
	        return false;
	      };
	    }
	  };
	});

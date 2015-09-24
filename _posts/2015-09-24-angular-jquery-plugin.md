---
layout: post
title: "在angular 中 使用jquery插件"
description: "在angular 中 使用jquery插件"
category: tech
tags: [angularjs,jquery]
---
{% include JB/setup %}

由于jquery的对dom处理的强大和悠久深入人心的历史，很多绚丽的效果不用jquery插件还真是麻烦，不过现在已经有很多插件都有专门的angularjs版本了。但是对于没有的，就只能自己包装一下了。其实不包装也是可以的，因为jquery本身就是全局的，变量都直接全局暴露了。但是在angular的管理和生命周期下，强烈觉得还是应该封装一下，让所有的页面元素都能被angular的体系所管理。

以我最喜欢的一款图片裁剪和伸缩放的插件[cropit](http://scottcheng.github.io/cropit/)为例。

在普通环境下：

	<div id="image-cropper">
	  <div class="cropit-image-preview-container">
	    <div class="cropit-image-preview"></div>
	  </div>
	  
	  <input type="range" class="cropit-image-zoom-input" />
	  
	  <input type="file" class="cropit-image-input" />
	  <div class="select-image-btn">Select new image</div>
	</div>
	
	<script>
		$('#image-cropper').cropit({
		  imageBackground: true,
		  imageBackgroundBorderWidth: 15 // Width of background border
		});
	</script>
	
在angular中这么用也没问题，但是会有卡顿。不是很清楚angular渲染和这个插件渲染的执行顺序。

插件可以封装成directive，以备复用。

直接摘抄我的代码：

	var plugin = angular.module('jqueryPluginDirective', []);
	
	// 图片上传裁剪处理插件
	plugin.directive('uploadCropImg',[function() {
	
	  return {
	    restrict: 'E',
	    replace: true,
	    scope: {
	      cropitConfig: "=",
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
			//...
	      };
	    }
	  };
	}]);
	
调用时：

	<upload-crop-img upload-imgdata="transferImgData" cropit-config = "cropitConfig"></upload-crop-img>
	
可以看出做了3件事：

1. 将html部分放到template里。
2. 将配置通过调用处的属性传入，通过设置scope接受。
3. 在link函数中初始化插件，`$(element).cropit(scope.cropitConfig);`
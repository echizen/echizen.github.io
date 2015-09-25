---
layout: post
title: "使用transclude在directive中获取子元素的控制权"
description: "使用transclude在directive中获取子元素的控制权"
category: tech
tags: [angularjs]
---
{% include JB/setup %}

有这样一段html:

	<div wx-upload-img attachment="goAuth.pageObject.dataGroup.image" img-status-tip="{{goAuth.pageObject.imgStatusTip}}" class="item item-icon-left mf_common_a flex flex_between" >
	   <div class="flex_item_3">证件审核</div>
      <div class="flex_item_6 align_right" ng-click="addImg()" >{{imgStatusTip}}</div>
    </div>
    
这里有一种需求，`wx-upload-img`是一个被多处复用的directive，他调用了微信上传图片的接口，但是每个复用他的地方ui都不一样，所以在这个directive中我们只控制功能，不控制ui，但是与它相关的ui元素需要绑定它提供的事件，使用它定义的元素。这样我们就要在`wx-upload-img`中获取它的子元素的控制权。

解决方案是使用`transclude`属性，获取isolate scope。

关键代码：

	transclude(scope, function(clone, scope) {
        elm.append(clone);
    });
    
完整示例：

	tool.directive('wxUploadImg',['wxUploadImg','PGgetImage',function(wxUploadImg,PGgetImage){
	  return {
	    restrict: 'EA',
	    replace: false,
	    scope: {
	      attachment : "=",
	      imgStatusTip: "@",
	    },
	    transclude:true,
	    link: function(scope, elm, attrs, ctrl ,transclude){
	      // 微信配置
	      /**
	       * 微信浏览器判断
	       * @return {Boolean} 
	       */
	      var isWechat =function(){
	          var ua = navigator.userAgent.toLowerCase();
	          if(ua.match(/MicroMessenger/i)=="micromessenger") {
	              return true;
	          } else {
	              return false;
	          }
	      };
		      
	      // 微信上传图片
	      var wxAddImg = function(){
	      	//...
	      };
	
		  // app上传图片
	      var appAddImg = function(){
	        //...
	      };
	
	      var saveImgToServer = function(url){
	        //...
	      };
	  
	      if(isWechat()){
	        scope.addImg = wxAddImg;
	      }else{
	        scope.addImg = appAddImg;
	      }
	      
	      transclude(scope, function(clone, scope) {
	        elm.append(clone);
	      });
	    }
	  };
	}]);
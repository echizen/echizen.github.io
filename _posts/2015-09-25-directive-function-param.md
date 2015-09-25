---
layout: post
title: "向directive中外部绑定的函数传递参数"
description: "向directive中外部绑定的函数传递参数"
category: tech
tags: [angularjs]
---
{% include JB/setup %}


适用情景：controller中定义的函数，通过属性传递绑定到directive的scope上，在directive内部处理后调用函数并传递参数。

示例：

controller:

	$scope.transferImgData = function(imageData){
      console.log(imageData);
      if(imageData.filePath === ''){
        SweetAlert.warning("请选择资讯图片，否则将使用默认资讯图片","");
      }else{
        feedbackMsg.uploadingImgMsg();

        imgUpload.post({},imageData,function(response) {
          
          if(response.code === 0){
            $scope.news.coverImage = response.data.fileUrl;
            feedbackMsg.uploadImgSuccessMsg();
          }else{
            console.log(response);
            SweetAlert.swal(response.message,"", "error");
          }
        });
      }
    };
    
view:

	<upload-crop-img upload-imgdata="transferImgData"></upload-crop-img>
	
directive:

	// 图片上传裁剪处理插件
	plugin.directive('uploadCropImg',[function() {
	
	  return {
	    restrict: 'E',
	    replace: true,
	    scope: {
	      uploadImgdata : "&"
	    },
	    template: '...',
	    link: function (scope, element) {
	    
	      scope.uploadImg = function(){
	      	    //要传输的图片数据
	            var imgData = {
	              'filePath':$('.cropit-image-input').val(),
	              'base64ImgData':$('.image-cropper').cropit('export')
	            };
	
	            // 执行传输图片数据的函数（由外部controller定义的函数）
	            scope.uploadImgdata()(imgData);
	      };
	    }
	  };
	}]);
	
`&`操作符保证了scope.uploadImgdata指代一个函数，scope.uploadImgdata()才是对这个函数的引用，等于controller中的$scope.transferImgData，然后需要传递参数。这里容易误解然后写成了`scope.uploadImgdata(imgData)`
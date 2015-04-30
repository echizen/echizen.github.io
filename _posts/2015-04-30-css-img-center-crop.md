---
layout: post
title: "css 居中裁剪图片、背景图片"
description: "css img center crop,css裁剪图片，居中裁剪图片、居中展示背景图片"
category: tech
tags: [css, basic]
---
{% include JB/setup %}

对于用户传的图片，或许没有按照要求的规格传，我们有时候会有按我们需要的规格展示的必要，这时候会为了美观裁剪，最科学的裁剪是居中裁剪。

通常我们会借助第三方插件裁剪，这时是真的将图片裁剪了才存储的。而css的“裁剪”只是展示时的选择性区域处理，并没有真正的裁剪原图片。

##背景图片展示

CSS

	i.icon.contactsLogo{
	    width: 100px;
	    height: 100px;
	    -webkit-transition-duration: 300ms;
	    transition-duration: 300ms;
	    background-position: center center;
	    background-size: cover;
	    background-repeat: no-repeat;
	    background-image: url('http://www.huiyouxing.com/image/meeting.png');
	    display: inline-block;
	    vertical-align: middle;
	}

html

	<div class="item-media">
	     <i class="icon contactsLogo contactsLogo4" id="logoPreview"></i>
	</div>

示例：
[http://jsfiddle.net/echizen/0oer38w8/2/](http://jsfiddle.net/echizen/0oer38w8/2/)


##图片展示

这个有缺陷，你得先知道你的图片是横屏还是竖屏模型（width>height还是width<height）。2种情况分别使用不同的样式，我还没有研究一种样式解决所有情况的。

CSS:

	.thumbnail {
	  position: relative;
	  width: 200px;
	  height: 200px;
	  overflow: hidden;
	}
	
	<!--width > height-->
	.thumbnail img {
	  position: absolute;
	  left: 50%;
	  top: 50%;
	  height: 100%;
	  width: auto;
	  -webkit-transform: translate(-50%,-50%);
	      -ms-transform: translate(-50%,-50%);
	          transform: translate(-50%,-50%);
	}
	
	<!--width < height-->
	.thumbnail img.portrait {
	  width: 100%;
	  height: auto;
	}
	

HTML:

	<!--width > height-->
	<div class="thumbnail">
	  <img src="landscape-img.jpg" alt="Image" />
	</div>
	
	<!--width < height-->
	<div class="thumbnail">
	  <img src="portrait-img.jpg" class="portrait" alt="Image" />
	</div>
	
示例：
[http://jonathannicol.com/blog/2014/06/16/centre-crop-thumbnails-with-css/](http://jonathannicol.com/blog/2014/06/16/centre-crop-thumbnails-with-css/)
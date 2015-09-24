---
layout: post
title: "网页跳转app逻辑"
description: "网页跳转app逻辑"
category: tech
tags: [js]
---
{% include JB/setup %}

经常会有从网页跳到app的需求，这时会有2种情况，app已下载能跳转，或者app未下载不能跳转/app已下载但因为某种原因跳转失败。这时用户怎么面对这个不成功的页面呢？

其实跳转app，是app中设置的一种协议，根据协议跳转，跳转后就不知道接下来的状态了，不会有跳转失败或者跳转成功的回调。而且跳转到app的时间也是不确定的。我们就只能设置一段合适的时间，这段时间后如果用户还停留在当前页，没有跳转到app，就认定他跳转失败，将页面导到设计的某个合适的页面，如下载页面。

	   var toUrl = 'hw.tech.JuHappy://enterMeeting/' + query.meetingId;
       location.href = toUrl;

       var downloadApp = setTimeout(function(){
          location.href = "http://dwz.cn/Ebiv9";
       },1000);
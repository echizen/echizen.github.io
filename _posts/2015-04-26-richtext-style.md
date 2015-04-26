---
layout: post
title: "富文本编辑器中内容在移动端适配问题"
description: "解决富文本编辑器中内容在移动端撑破页面"
category: tech
tags: [css, basic]
---
{% include JB/setup %}

我们经常遇到一种场景，管理者在电脑端后台的文本编辑器下编辑了一段内容，然后要在手机端给用户展现出这段内容。譬如说最近开发的会友行的会议详情部分，手机端会议内容部分包括时间地点、人数等等，还有富文本编辑器内的会议详情部分，这部分尝尝包含图片、表格等多重样式，在手机端如果不把这部分内容的容器的样式设置好，经常会破相，出现滚屏啦、内容被截啦。

在容器div下加上这些样式就可以避免这个问题了。


	.rich_media_content {
	  position: relative;
	  overflow: hidden;
	}
	
	.rich_media_content * {
	  max-width: 100%!important;
	  box-sizing: border-box!important;
	  -webkit-box-sizing: border-box!important;
	  word-wrap: break-word!important;
	}
	
	.rich_media_content table {
	    margin-bottom: 10px;
	    border-collapse: collapse;
	    display: table;
	    width: 100%!important
	}
	
	.rich_media_content td,th {
	    word-wrap: break-word;
	    word-break: break-all;
	    padding: 5px 10px;
	    border: 1px solid #DDD
	}
	
	.rich_media_content caption {
	    border: 1px dashed #DDD;
	    border-bottom: 0;
	    padding: 3px;
	    text-align: center
	}
	
	.rich_media_content th {
	    border-top: 2px solid #BBB;
	    background: #f7f7f7
	}
	
	.rich_media_content td p {
	    margin: 0;
	    padding: 0
	}
	
	.rich_media_content form {
	    display: none!important
	}
	
	@media screen and (min-width: 0\0) and (min-resolution:72dpi) {
	    .rich_media_content table {
	        table-layout:fixed!important
	    }
	
	    .rich_media_content td,.rich_media_content th {
	        width: auto!important
	    }
	}
	
	.rich_media_content .tc {
	    text-align: center
	}
	
	.rich_media_content .tl {
	    text-align: left
	}
	
	.rich_media_content .tr {
	    text-align: right
	}
	
html结构：

	<div class="rich_media_content">
      	<div>{{detailContent}}</div>
  	</div>
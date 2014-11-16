---
layout: post
title: "js 循环给html中添加含动态数据的dom元素"
description: "js 循环给html中添加含动态数据的dom元素，js模板制作，后台数据插入到前端页面"
category: tech 
tags: [前端, js, html, 数据交互, 进阶, tutorial]
---
{% include JB/setup %}
#应用场景
前端经常在如ajax交互中将后台数据动态更新插入到dom中，而且插入的部分多是循环元素，只是数据不同，如做留言板是，每个评论者评论后评论模块的更新。

#函数
-  html():可以获取一段html文档（注意不是副本），但不是dom元素，不能用针对dom元素的函数如find()对其进行操作。
-  clone():拷贝一段dom节点的副本，可以用对dom操作的函数对其进行操作，如parent().
-  replace(reg,content):正则替换,将reg正则表达式匹配的内容用content元素替换。

**注**：用html()还是clone()就看你对模板的操作需求，如果你只是对每个模板替换掉动态显示的数据部分，用html就够了，如果你还要为每个模板操作一些特别的dom部分，如添加一段html或更改一段html部分，建议用clone()函数。


#思路

将要重复性插入的dom元素制成模板，在页面中`display:none`后，用`html()`或`clone()`函数拷贝一份，然后用replace函数将动态数据部分进行替换。在`append()`或`prepend()`或`after()`或`before()`等等插入到页面中。

#示例

##1.弄清需求
	<div class="col-xs-12 formItem_wrap" >
        <h4>
            <span class="glyphicon glyphicon-plus" style="color:#fca837"></span>
            添加新表单项
        </h4>
        <div class="col-xs-3 " data-elementType="8" data-type="0" >
            <div class="col-xs-12 formItem" data-fieldname="">日期</div>
        </div>
        <div class="col-xs-3 " data-elementType="9" data-type="0" >
            <div class="col-xs-12 formItem" data-fieldname="">时间</div>
        </div>
    </div>
    
 要在`.formItem_wrap`中循环输出：
 
        <div class="col-xs-3 " data-elementType="9" data-type="0" >
            <div class="col-xs-12 formItem" data-fieldname="">时间</div>
        </div>
        
 只是每个循环体的`data-elementType`、`data-type`、`data-fieldname`以及文字描述都不一样。
 
##2. 制作模板
 
    <div id="form_list_template" style="display:none">
         <div class="col-xs-3 " data-elementType={$elementType} data-type="1">
              <div class="col-xs-12 formItem" data-fieldname={$fieldname}>{$itemLabel}</div>
         </div>
    </div>
    
##3.操作模板

	//要替换模板内容的json数组
	var fixedFields = [
	        {"itemLabel": "姓名", "field":"realname", "tvalues":"null", "elementType":"0"},
	        {"itemLabel": "性别", "field":"sex", "tvalues":"保密|,|男|,|女", "elementType":"11"},
	        {"itemLabel": "生日", "field":"birthday", "tvalues":"null", "elementType":"8"},
	        {"itemLabel": "住址", "field":"address", "tvalues":"null",  "elementType":"0"},
	        {"itemLabel": "邮箱", "field":"email", "tvalues":"null","elementType":"3"},
	        {"itemLabel": "手机", "field":"mobile", "tvalues":"null", "elementType":"7"},
	        {"itemLabel": "电话", "field":"phone", "tvalues":"null", "elementType":"6"},
	        {"itemLabel": "qq", "field":"qq", "tvalues":"null",  "elementType":"2"},
	        {"itemLabel": "微博", "field":"weibo", "tvalues":"null", "elementType":"0"},
	        {"itemLabel": "证件类型", "field":"idType", "tvalues":"身份证|,|警官证|,|学生证|,|结婚证|,|驾驶证", "elementType":"14"},
	        {"itemLabel": "证件号码", "field":"idNumber", "tvalues":"null", "elementType":"0"},
	        {"itemLabel": "婚姻", "field":"married", "tvalues":"未婚|,|已婚|,|离异|,|丧偶", "type":"0", "elementType":"10"},
	
	    ];
	    
	    //获取模板
	    var form_list_template = $("#form_list_template").html();
	    
	    //操作模板
	    var form_item_init=function(){
	        var form_list ;
	        for ( key in fixedFields){
	            //由于html()不是克隆，而是获取，为了不破坏原模板，我们将原模板拷贝给另一个变量，然后操作这个变量。
	            form_list = form_list_template;
	            form_list = form_list.replace(/\{\$elementType\}/gi,fixedFields[key].elementType);
	            form_list = form_list.replace(/\{\$type\}/gi,fixedFields[key].type);
	            form_list = form_list.replace(/\{\$itemLabel\}/gi,fixedFields[key].itemLabel);
	            form_list = form_list.replace(/\{\$fieldname\}/gi,fixedFields[key].field);
	            //将改造后的html输出到页面
	            $('.formItem_wrap').append(form_list);
	        }
	
	    }

       form_item_init();

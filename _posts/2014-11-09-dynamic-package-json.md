---
layout: post
title: "动态json创建"
description: "动态json创建,将dom中的数据动态添加到"
category: tech
tags: [ 前端, js, 数据交互, tutorial]
---
{% include JB/setup %}

# 动态json创建

## 前言
作为一个前端，免不了要与后台攻城狮在页面上进行数据交互，话说这次的交互数据封装传输的任务都在我这边我也是吐了，用可怜的jquery在那儿dom->数据、数据->dom 整合，还要打包成json，再传给后台，我也是醉了，所以赶完这个项目我就去学angular.js了，数据和dom的交互jquery展现的性能太弱了，还是mvc的框架-angular.js强大。


## 精粹

json的动态创建其实就是用了push函数。
json数据格式要注意几点：

1.  json的键值对必须先定义后使用。就是说你可以定义一个
`jsonData={"name":"","age":""}`，再通过`jsonData.name="marry"`给这个对象赋值，但是你不能直接在未定义`jsonData`格式的情况下，就用`jsonData.name="marry"`的形式给它赋值。（这样你也许就纳闷了，因为多数情况下你不可能对一个很长的json数据先定义，所以我们要用push函数）

2. json数据访问时`jsonData.name`中的name是键的名字，只能是字符串，如果你要获取一个变量key代表的键，要用jsonData[key]的方式获取。

   现在举个列子：

		var prizer = {
            first: "huhu",
            second: "lala",
        };
        var key = "first"
        prizer.key
		=> undefined
		prizer[key]
		=> "huhu"
        
	  你要获得prizer底下键为first的值，直接用`prizer.key`只会报`undefined`的错误，因为它把key当成键`“key”`了，这事用`prizer[key]`可以获取"huhu"的值。
  
3.  对于一个json数组，形式类似`[{...},{...},{...}]`，动态创建：
	
		var jsonArray = [];
		jsonArray.push({...});

4.  对于一个json对象，形式类似`{ "a":[{...},{...}], "b":[{...},{...},{...}], "c":[{...}] }`,动态创建：

		var json = {};
		json.a=[];
		json.a.push({...});


以下实例内容没有整理，只是贴了自己写过的代码作为笔记，看懂上面内容后可忽略以下内容。
## 实例：json数组动态创建
假设你要封装一个如下所示的json：

	[
	  {
	     "itemLabel":"姓名",
	     "field":"realname",
	     "tvalues":[],
	     "priority":0,
	     "elementType":"0"
	  },
	  {
	     "itemLabel":"性别",
	     "field":"sex",
	     "tvalues":[
	                  {"oId":"","text":"保密"},
	                  {"oId":"","text":"男"},
	                  {"oId":"","text":"女"}
	               ],
	      "priority":1,
	      "elementType":"11"
	  },
	  {
	      "itemLabel":"生日",
	      "field":"birthday",
	      "tvalues":[],
	      "priority":2,
	      "elementType":"8"
	  },
	  {
	      "itemLabel":"证件类型",
	      "field":"idType",
	      "tvalues":[
	                    {"oId":"","text":"身份证"},
	                    {"oId":"","text":"警官证"},  
	                    {"oId":"","text":"学生证"},  
	                    {"oId":"","text":"结婚证"},
	                    {"oId":"","text":"驾驶证"}
	                 ],
	      "priority":3,
	      "elementType":"14",
	      "ext":"0"
	  },
	  {
	      "itemLabel":"标题",
	      "field":"",
	      "tvalues":[
	                    {"oId":"","text":"选项A","img":"../img/undefine_pic.png"},
	                    {"oId":"","text":"选项B","img":"../img/undefine_pic.png"}
	                 ],
	      "priority":4,
	      "elementType":"16",
	      "ext":"4"
	   }
	]

这是一个json数组，只是我只取了特殊的部分，只取了下标为0的部分，其他类似。

这个json最多有3层。

js动态生成：（这么贴代码是为了给自己留个笔记，嫌太长的请自动忽略。）

    var get_formdata = function(obj,type){
        data_area = obj;
        var formdata = [];

        data_area.find('.js_userform_wrap .input_row .form-group').each(function(index){
            var itemLabel =  $(this).children('.itemLabel').text();
            itemLabel = itemLabel.substring(0, itemLabel.length - 1);
            var elementType = $(this).children('input[name="elementType"]').val();
            var ext = $(this).children('input[name="ext"]').val();
            if(type == 'edit'){
                var sIId = data_area.find('input[name="itemId"]').val();
                formdata.push({"sIId":sIId, "itemLabel":itemLabel, "field":"", "tvalues":[],  "priority":index, "elementType":elementType, "ext":undefined});
            }else{
                var sIId = "";
                formdata.push({ "itemLabel":itemLabel, "field":"", "tvalues":[],  "priority":index, "elementType":elementType, "ext":undefined});
            }
            
            if($(this).parent('.input_row').data('itemtype')=='textarea_input'){

                var field =  $(this).find('.js_itemName_wrap textarea').attr('name');
                var tvalues = "";//$(this).find('.js_itemName_wrap input').val();
                formdata[index].tvalues = [];

            }else if($(this).parent('.input_row').data('itemtype')=='select_input'){
                
                var select = $(this).find('.js_itemName_wrap select');
                var field = select.attr('name');
                select.find('option').each(function(i){

                    var oId = $(this).parent().siblings('.oId_wrap').find('input[name="oId"]:eq('+i+')').val();
                    if(oId=='' || oId == 'undefined' || oId== '{$oId}' ){
                        oId = '';
                    }
                    var text = $(this).text();
                    formdata[index].tvalues.push({"oId":oId,"text":text});
                })
                
            }else if($(this).parent('.input_row').data('itemtype')=='radio_input'||$(this).parent('.input_row').data('itemtype')=='check_input'){
                var field =  $(this).find('.js_itemName_wrap input[type!="hidden"]').attr('name');
                var radio = $(this).find('.js_itemName_wrap .js_choice_content');
                radio.each(function(){
                    var oId = $(this).children('input[name="oId"]').val();
                    if(oId=='' || oId == 'undefined' || oId== '{$oId}'){
                        oId = '';
                    }
                    var text = $(this).find('label span').text();
                    formdata[index].tvalues.push({"oId":oId,"text":text});
                })
                if($(this).parent('.input_row').data('itemtype')=='check_input'){
                    var ext = $(this).parent('.input_row').find('.js_ext').children('option:selected').val();
                    formdata[index].ext = ext;
                }
                
            }else if($(this).parent('.input_row').data('itemtype')=='radioImg_input'||$(this).parent('.input_row').data('itemtype')=='checkImg_input'){

                var field = $(this).find('.js_itemName_wrap input[type!="hidden"]').attr('name');
                var radio = $(this).find('.js_itemName_wrap .js_choice_content');
                radio.each(function(){
                    var oId = $(this).children('input[name="oId"]').val();
                    if(oId=='' || oId == 'undefined' || oId== '{$oId}'){
                        oId = '';
                    }
                    var img = $(this).find('label img').attr('src');
                    formdata[index].tvalues.push({"oId":oId,"text":img});
                })

                if($(this).parent('.input_row').data('itemtype')=='checkImg_input'){
                    var ext = $(this).parent('.input_row').find('.js_ext').children('option:selected').val();
                    formdata[index].ext = ext;
                }

            }else if($(this).parent('.input_row').data('itemtype')=='radioTextImg_input'||$(this).parent('.input_row').data('itemtype')=='checkTextImg_input'){
                var field = $(this).find('.js_itemName_wrap input[type!="hidden"]').attr('name');
                var radio = $(this).find('.js_itemName_wrap .js_choice_content');
                radio.each(function(){
                    var oId = $(this).children('input[name="oId"]').val();
                    $(this).children('input[name="oId"]').attr({'data-oId':oId});
                    if(oId=='' || oId == 'undefined' || oId== '{$oId}'){
                        oId = '';
                    }
                    var text = $(this).find('label span.js_choice_name').text();
                    var img = $(this).find('label img').attr('src');
                    formdata[index].tvalues.push({"oId":oId,"text":text,"img":img});
                    
                })
                if($(this).parent('.input_row').data('itemtype')=='checkTextImg_input'){
                    var ext = $(this).parent('.input_row').find('.js_ext').children('option:selected').val();
                    formdata[index].ext = ext;
                }
            }else if($(this).parent('.input_row').data('itemtype')=='score_input'){
                var field =  $(this).find('.js_itemName_wrap').attr('name');
                formdata[index].tvalues = []; 
            }else{//text_input/data_input/time_input
                var field =  $(this).find('.js_itemName_wrap input').attr('name');
                formdata[index].tvalues = [];
            }
            formdata[index].field = field;
        })
        return formdata;
    }

这么贴代码是为了给自己留个笔记。

## 实例：json对象动态创建

要封装成：

	   var prizer = {
			             'first': [{}],
			             'second': [{}],
			             'third': [{}],
			        };

以下代码中生成`config.get`是获取之前函数中的返回值填充到prizer中。`dataSource[nums[i]]`为json对象。（这是个代码片段。。。）

		el.result = function(){
		            var prizer = {};
		            keycode = ['first','second','third'];
		            for(key in keycode){
		                var awards = keycode[key],//first\second\third
		                    locals = config.get(awards);//字符串
		                    prizer[awards] = [];

		                if( locals ){
		                    var nums = locals.split(',');//nums为数组形式
		                    for(var i = 0; i < nums.length; i++){
		                        prizer[awards].push(dataSource[nums[i]]);
		                    }
		                }
		            }
            return prizer ;
        }
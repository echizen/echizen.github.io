---
layout: post
title: "跨域通信"
description: ""
category:tech
tags: [html5, 前端, 进阶]

---
{% include JB/setup %}
#跨域通信
网上关于跨域通信的例子不少，方法也很多，强烈建议用html5 的communication API。

##原由：同源策略
浏览器为了web安全，制定了同源策略，主要针对以下情况的网页之间不能通信。

+ 不同域名

* 不同端口

+ 不同协议

##可以跨域的元素
   在浏览器中，`<script>`、`<img>`、`<iframe>`和`<link>`这几个标签是可以加载跨域（非同源）的资源的，并且加载的方式其实相当于一次普通的GET请求，唯一不同的是，为了安全起见，浏览器不允许这种方式下对加载到的资源的读写操作，而只能使用标签本身应当具备的能力（比如脚本执行、样式应用等等）。

##html5 跨域：postMessage

这是一套新规范，从我个人使用角度而言，推荐他的原因是，他简洁、安全、可即时通信。
常用他来进行iframe父子页面的通信。

###示例:www.child.com要向www.parent.com传输信息。

其中www.child.com是www.parent.com中嵌套的iframe子页面的源.

在www.parent.com页面中添加message事件监听器，监听子页面www.child.com传来的值

		window.addEventListener("message", function( event ) { 
            // 判断数据来源是否是信任的源
            if(event.origin=="https://www.child.com"){
              // 把子窗口发送过来的数据显示在父窗口中
               $("#chosed_userform").val(event.data.form_id);
               $('#userform_iframe').removeClass('active');
               $("#chosed_outcome").text("您选择的是"+event.data.form_name);
            }
        }, false );  
        
在www.child.com页面添加postMessage方法，向父页面www.parent.com传递信息。    
        
     //操作父元素
    function sendId(){ 
        // 通过 postMessage 向父窗口发送数据
        var form_id = $('input[name="check_userform"]:checked').val();
        var form_name = $('input[name="check_userform"]:checked').parent("label").text();
        var checked_form = {
            "form_id": form_id,
            "form_name": form_name,
        }
        window.parent.postMessage( 
                checked_form, 
                "https://www.parent.com"
            );
    } 
    
    
   类似也可以由父页面向子页面发送数据。是不是很简洁。
   
###postMessage API

1. 发送消息：postMessage(data,goal_origin)
	example:`Window.postMessage("hello","https://www.a.com")`
	在iframe中一般在contentWindow对象下调用该函数，如：
	`document.getElementsByTagName("iframe").contentWindow.postMessage("hello","https://www.a.com")`
	 	
2. 监听消息message

   ` window.addEventListener("message", handleFunction(e),true/false)`
   handleFunction(e)是处理函数，参数`e.origin`是消息来源的URL，`e.date`是传来的消息内容。
   第三个参数是定义事件传递顺序了，true使用capture，是由父元素逐步传递到子元素，false使用bubbling，从子元素逐步传递到父元素，就是常说的冒泡。


##网传其他跨域方式

1. **服务器端代理**，缺点在于，默认情况下接收Ajax请求的服务端是无法获取到的客户端的IP和UA的。

2. **iframe**，使用iframe其实相当于开了一个新的网页，具体跨域的方法大致是，域A打开的母页面嵌套一个指向域B的iframe，然后提交数据，完成之后，B的服务端可以：
 	+ 返回一个302重定向响应，把结果重新指回A域；
	+ 在此iframe内部再嵌套一个指向A域的iframe。
	
	这两者都最终实现了跨域的调用，这个方法功能上要比下面介绍到的JSONP更强，因为跨域完毕之后DOM操作和互相之间的JavaScript调用都是没有问题的，但是也有一些限制，比如结果要以URL参数传递，这就意味着在结果数据量很大的时候需要分割传递，甚是麻烦；还有一个麻烦是iframe本身带来的，母页面和iframe本身的交互本身就有安全性限制。

3. **JSONP**：利用script标签跨域，这个办法也很常见，script标签是可以加载异域的JavaScript并执行的，通过预先设定好的callback函数来实现和母页面的交互。它是一个非官方的协议，这个callback函数，对它的使用有一个典型的方式，就是通过JSON来传参，即将JSON数据填充进回调函数，这就是JSONP的JSON+Padding的含义。

	在互联网上有很多JSONP的服务来提供数据，本质上就是跨域请求，并且在请求URL中指定好callback，比如callback=result，那么在获取到这些数据以后，就会自动调用result函数，并且把这些数据以JSON的形式传进去，例如（搜索“football”）：

	http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=football&callback=result

	使用JQuery来调用就写成：

		$.getJSON("http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=football&callback=?",function(data){
		    //...
		});
	总的来说，JSONP的跨域方式的局限性在于，只能使用GET请求，并且不能解决不同域的两个页面之间如何进行JavaScript调用的问题。

4.**Access Control**：
	有一些浏览器支持Access-Control-Allow-Origin这样的响应头，比如：`header("Access-Control-Allow-Origin: http://www.a.com");`
就指定了允许对www.a.com跨域访问。

6.**window.name**：这个东西其实以前被用作黑客XSS的手段，其本质是，当window的location变化的时候，页面会重新加载，但是有趣的是，这个window.name居然不发生变化，那么就可以用它来传值了。配合iframe，改变几次iframe的window对象，就完成了实用的跨域数据传递。

5.**document.domain**：这个方式适用于a.example.com和b.example.com这种跨域的通信，因为二者有一个共有的域，叫做example.com，只要设置document.domain为example.com就可以了，但是如果a.example1.com和b.example2.com之间要通信，它就没办法了。










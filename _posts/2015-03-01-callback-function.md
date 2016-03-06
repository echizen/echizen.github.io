---
layout: post
title: "js回调函数"
description: "js回调函数"
category: tech
date: 2015-01-24 21:02:32
tags: [pro, js, 前端]
---
{% include JB/setup %}



感觉这是我写不好的一篇博客，本人认识实在是太浅了，最近才意识到回调函数。

## 感受回调函数
 
其实我们使用回调函数的场景蛮多的，不过我们不一定意识到我们在使用回调函数。

### 场景一：插件回调函数使用

eg:

    var n =  $('#noteContainer').noty({
        layout      : 'center',
        closeWith   : ['click'],
        theme       : 'relax',
        maxVisible  : 20,
        timeout: 2000,
        animation   : {
            open  : 'animated flipInX',
            close : 'animated flipOutX',
            easing: 'swing',
            speed : 500
        },
        callback: {
            onShow: function() {
                $('#noteContainer').css({'pointer-events':'auto'});
            },
            afterClose: function() {
                $('#noteContainer').css({'pointer-events':'none'});
            },
        },
    });
    
 这里的onshow和afterClose函数在插件内部就是当做回调函数使用的。一般插件都会有callback API，那就是回调函数接口。
 
### 场景二：jquery函数

譬如说事件绑定：

	$('#coverImageContainer').on('click','.cover_image',function(){
    	showModal($(this));
    });
    
譬如说ajax请求，参数有回调函数：

	$.post('',formData,function(result){
        if (result.errorCode == 0) {
            generate('success', result.errorMessage);
        }else{
            $('#save').removeAttr('disabled');
        };
    },'json');
    
再譬如说某些内部函数如`each`，它的参数也是回调函数:

	$('.cover_image').each(function(){
	    var thisId = $(this).attr('data-imgId');
	    if(id == thisId){
	        $(this).remove();
	        return false;
	    } 
	})
	
## 什么是回调函数：

**回调函数是一个被作为参数传递给另一个函数的函数**。从定义来讲，回调函数一点也不神秘，他就是其他函数的参数，只不过它这个参数本身就是函数。

## 回调函数独特之处

因为函数在Javascript中是第一类对象，我们像对待对象一样对待函数，因此我们能像传递变量一样传递函数，在函数中返回函数，在其他函数中使用函数。当我们将一个回调函数作为参数传递给另一个函数是，我们仅仅传递了函数定义。我们并没有在参数中执行函数。我们并不传递像我们平时执行函数一样带有一对执行小括号()的函数。

需要注意的很重要的一点是回调函数并不会马上被执行。它会在包含它的函数内的某个特定时间点被“回调”，这个特定时间点因函数而异。

### 回调函数是闭包

我们将一个回掉函数作为变量传递给另一个函数时，这个回掉函数在包含它的函数内的某一点执行，就好像这个回调函数是在包含它的函数中定义的一样。这意味着回调函数本质上是一个闭包。

正如我们所知，闭包能够进入包含它的函数的作用域，因此**回调函数能获取包含它的函数中的变量**，以及全局作用域中的变量。

因此，你不能随便给回调函数传参，否则会发生undefined的错误。

	function callTwoFun(fun){
	    fun();
	}
	function oneFun(param){
	    callTwoFun(function(param){
	        console.log(param);
	    })
	}

在这里，我们向callTowFun传递了回调函数作为参数，且`callTwoFun(function(param){
	        console.log(param);
	    })`里的function的写法只是定义，不是执行。我们给这个回调函数声明了形参param，但是这种写法，我们却无法在执行时传递param实参。只会抛出`undefined`.
	    
正确的写法应该是：

	function callTwoFun(fun){
	    fun();
	}
	function oneFun(param){
	    callTwoFun(function(){
	        console.log(param);
	    })
	}

不用担心回调函数中的param取不到值，我们已经解释过了，回调函数能获取包含它的函数中的变量，也就是callTowFun能获取param参数，那么它的回调函数也能获取这个参数。

另外你可能疑惑`callTwoFun`的定义中，执行fun()时并没有携带参数，要知道**在js中形参有映射的argument参数，所以我们在掉用一个带有形参的函数时，即可以不传递参数，也可以传递多于形参数量的实参，argument会按照实际情况处理**。

## 回调函数使用注意

### 在执行之前确保回调函数是一个函数

如果没有适当的检查，如果参数中没有一个回调函数或者传递的回调函数事实上并不是一个函数，我们的代码将会导致运行错误。虽然这不是必须的，但是作为良好习惯，我们应该处理这种错误。

上面的例子应加上判断：

	function callTwoFun(fun){
	    if(typeof fun === "function"){
        	//调用它，既然我们已经确定了它是可调用的
            fun();
    	}
	}


## 避免回调地狱

虽然回调函数有很多方便之处，好吧，我没有提他的优点。。。但是回调函数要是层层嵌套太多，增加了阅读和代码维护成本，是不可取的。

譬如，你能读懂下面这段的代码的意境吗。。。这可是node-mongodb-native，一个适用于Node.js的MongoDB驱动中的一个例子。

	 var p_client = new Db('integration_tests_20', new Server("127.0.0.1", 27017, {}), {'pk':CustomPKFactory});
	    p_client.open(function(err, p_client) {
	        p_client.dropDatabase(function(err, done) {
	            p_client.createCollection('test_custom_key', function(err, collection) {
	                collection.insert({'a':1}, function(err, docs) {
	                    collection.find({'_id':new ObjectID("aaaaaaaaaaaa")}, function(err, cursor) {
	                        cursor.toArray(function(err, items) {
	                            test.assertEquals(1, items.length);
	
	                            // Let's close the db
	                            p_client.close();
	                        });
	                    });
	                });
	            });
	        });
	    });

## 福利

又一个例子，我的入门案例：[http://jsfiddle.net/echizen/pfucjhck/](http://http://jsfiddle.net/echizen/pfucjhck/)

一篇写的不错的博客，我的入门导读：[http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/](http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/)



  
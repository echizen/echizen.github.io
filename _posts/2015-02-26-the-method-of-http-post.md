---
layout: post
title: "从angularJs的ajax发觉post 请求方式"
description: "the method of http post，php端如何接收angularjs的ajax请求，angularjs的ajax请求方式，post请求的方式，php接收不同post请求的方法"
category: tech
tags: [pro, angularJs, php, 前端]
---
{% include JB/setup %}

## 缘由

前端：angularJs发出ajax请求，提交表单数据。

	$scope.remove = function(index){
	    var sureDel=confirm("确定删除该商品？");
	    if (sureDel==true){
	        var  deleteData = { cartId:$scope.items[index].cartId};
	        $http.post("{:U('App/Order/delCartGoods')}",deleteData).success(function(result){
	            console.log(result);
	            if(result == '1'){
	                $scope.items.splice(index,1);
	            }else{
	                alert('删除失败，请再次操作');
	            }
	        }).error(function(){
	            alert('删除失败，请再次操作');
	        });
	    }
	}
	
后台：php，ThinkPHP框架

	<?
		$id = $_POST['cartId'];
	?>
	
结果php取到的$id为空。查看网络请求，chrome下的控制台显示`Request Method:POST
Status Code:200 OK`,`Request Method:POST
Status Code:200 OK`，数据发送出去了啊，发送之前打印一遍也是有值的。

仔细把header看了一遍，发现`Content-Type:application/json`,平时用form直接提交数据是`Content-Type:application/x-www-form-urlencoded`。google一下，才知道post请求也有4种方式。

## post请求的4种方式

### application/x-www-form-urlencoded

普通的表单提交，如果不设置enctype 属性，都会以这种方式提交。

	POST http://www.example.com HTTP/1.1
	Content-Type: application/x-www-form-urlencoded;charset=utf-8
	 
	title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
	
提交的数据按照 key1=val1&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码。大多数常用框架如jquery的ajax请求方式就是这种方式。

### application/json

这种方式用来告诉服务端消息主体是序列化后的 JSON 字符串。

	POST http://www.example.com HTTP/1.1
	Content-Type: application/json;charset=utf-8
	 
	{"title":"test","sub":[1,2,3]}
	
angularJs的Ajax默认就是这种方式。这种方式php 就无法通过 $_POST 对象从上面的请求中获得内容。

### multipart/form-data

我们使用表单上传文件、图片时，会设定让form 的 `enctyped=multipart/form-data`，其实就是声明用这种方式提交数据。

### text/xml

它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范。常用作搜索引擎的 ping 服务，不过个人认为格式比较繁琐，没用过。


## php接收以application/json方式post的数据

### 从 php://input 里获得原始输入流再 json_decode 成对象

	<?
		$postdata = file_get_contents("php://input");
		$request = json_decode($postdata);
	?>
	
### 声明header

	<?
		header('Content-Type: application/json');
		$request = json_encode($_POST);
	?>
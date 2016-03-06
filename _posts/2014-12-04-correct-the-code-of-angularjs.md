---
layout: post
title: "《AngularJs》代码纠错"
description: "correct the code of <<AngularJs>>"
category: tech
tags: [tutorial,基础,AngularJs]
---
{% include JB/setup %}

angulajs入门，用的是O'REILLY的《AngularJs》一书，这是一本好书，书中解释的很到位，例子也都很典型。但是这是一本13年的书，规范在不断发展，所以书中有些代码的写法已被遗弃，按照书中的代码来写会报错，将我发现解决的错误记下来，以免其他初学者也出错。

![image](https://echizen.github.io/assets/blog-img/QQ20141204-1.png)


## 1. page6
原：

{% raw %}

	<html ng-app="myapp">
	    <head>
	        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	        <title>示例</title>
	        <script type="text/javascript" src="../js/angular.min.js"></script>
	    </head>
	    <body ng-controller='CartController'>
	        <h1>购物车</h1>
	        <div ng-repeat='item in items'>
	            <span>{{item.title}}</span>
	            <input ng-model='item.quantity'>
	            <span>{{item.price | currency}}	            </span>
	            <span>{{item.price * item.quantity | currency}}</span>
	            <button ng-click="remove($index)">Remove</button>
	        </div>
	        
	        <script type="text/javascript">
	            
	                $scope.items = [
	                                    {title:'paint pots',quantity:8,price:3.95},
	                                    {title:'polka dots',quantity:17,price:12.95},
	                                    {title:'pebbles',quantity:5,price:6.95},
	                                ];
	
	                $scope.remove = function(index){
	                    $scope.items.splice(index,1);
	                }
	        </script>
	    </body>
	</html>

{% endraw %}	
	
修改的地方：

		<script type="text/javascript">
            var myAppModule = angular.module('myapp', []);
            myAppModule.controller('CartController',function($scope){
                $scope.items = [
                                    {title:'paint pots',quantity:8,price:3.95},
                                    {title:'polka dots',quantity:17,price:12.95},
                                    {title:'pebbles',quantity:5,price:6.95},
                                ];

                $scope.remove = function(index){
                    $scope.items.splice(index,1);
                }
            })
        </script>
        
原因：新的版本要求必须用`angular.module('appname', []);`这种形式初始化自己定义的`ng-app='appname'`，如果你直接写'ng-app'而不去初始化自己的名字，则不用加这句。不过在单文本模式下，显然我们可能会一个页面多个作用域，也就是多个ng-app,不定义名称，显然不是好的做法。


## 2. page23
原:

			function DeathrayMenuController($scope){
                $scope.menuState.show = false;

                $scope.toggleMenu = function(){
                    $scope.menuState.show = !$scope.menuState.show;
                }
            }
            
改为：

			function DeathrayMenuController($scope){
                $scope.menuState = {'show':false};

                $scope.toggleMenu = function(){
                    $scope.menuState.show = !$scope.menuState.show;
                }
            }

原因：典型的未定义就使用。

## 3. page30
原：

	$scope.subtotal = function(){
        return $scope.totalCart() - $scope.discount;
    }
    
改：

	$scope.subtotal = function(){
        return $scope.totalCart() - $scope.bill.discount;
    }

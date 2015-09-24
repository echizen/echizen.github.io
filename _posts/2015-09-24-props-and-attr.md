---
layout: post
title: "jquery中props和attr的不同"
description: "jquery中props和attr的使用场景，不同功能"
category: tech
tags: [jquery,js]
---
{% include JB/setup %}

又是一段long long time没写博客了，要写的东西又堆积如山了。。。

我们经常会有在代码中改变checkbox的checked属性的需求，如全选这种，也有要及时获取checkbox的checked属性值的需求。可能会遇到：

	<input type="checkbox" />
	<script type="text/javascript">
        $('#checkbox').click(function() {
            console.log($(this).attr('checked'));
            console.log($(this).prop('checked'));
        });
    </script>
    
   点击checkbox,你会发现打印的是：undefined  、 true
   					           
   再点击checkbox：  undefined  、 false。
   
   可以看出，`attr`取出的是checkbox一开始的checked属性的值，不会及时改变，而`prop`取得是动态值，会随着checkbox状态改变。
   
   
   **具有 true 和 false 两个值的属性，如 checked, selected 或者 disabled 使用prop()获取值，其他的使用 attr()**
   
   再来看看jquery源码中这2个函数的实现方式：
   
  - prop: 
  
    	return elem[ name ];
		eg:document.getElementById('checkbox').checked = true||false。这是原生js提供的接口。
		
		
 - attr:
		
		ret = jQuery.find.attr( elem, name );
        // Non-existent attributes return null, we normalize to undefined
        return ret == null ? undefined : ret;
		elem.getAttribute( name )
		

看本质还是js的getAttribute方法取到的是初始值，而dom api中elem.attr方法取到的是及时值，会在调用时去扫描elem的attr及时的值。
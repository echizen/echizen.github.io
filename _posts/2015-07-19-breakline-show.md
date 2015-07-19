---
layout: post
title: "html解析换行符"
description: "html解析换行符"
category: tech
tags: [html, basic]
---
{% include JB/setup %}

遇到简单的textarea用户输入内容的展示时，总是避免不了要展示换行符。而换行符会随其他数据一起存储在服务器中。但是在源码中是看不到的。

![image](https://echizen.github.io/assets/blog-img/AE4DF29E-874D-46DB-822E-95BC69768547.png)

在测试中发现换行符是存在的，只是没有在代码里显示出来而已，存在于字符数组中，可以通过charCodeAt函数取出看到它的值是10，查一下10是换行符`/n`的编码。

知道这些就可以用正则匹配来实现换行了。

 	(notifications[i]['messageBody']).replace(/\r?\n/g,'<br/>')
---
layout: post
title: "浏览器的刷新和缓存"
description: "浏览器的刷新和缓存机制，回车、f5和ctrl+f5浏览器都做了什么(mac上回车、cmd+r和cmd+shift+f5)"
category: tech
tags: [浏览器, 缓存]
---
{% include JB/setup %}

这个问题来源于一次再IE上调试时，先上传了一份无bug版本用ie打开后正常，接下来不小心上传了一份bug版本，偶然发现浏览器回车刷新页面看到的是正常版（缓存），按刷新按钮和f5看到是Bug版。不禁心生疑惑。


浏览器向服务器发送请求时是会带上头信息的，用来告诉服务器一些附加信息，其中一些参数与缓存有关，这些信息再chrome中的network中可以看到。

# 回车、cmd+r和cmd+shift+f5，浏览器都发了什么头信息

对于一个网址中的js资源（我是拿正打开的csdn页面测试的）。

### 地址栏回车

1. status code:304

2. request header:

   ![image](https://echizen.github.io/assets/blog-img/blog2016030501.jpg)
 
3. 特点：`if-modify-since`

### 刷新按钮/cmd+R(windows为f5)

1. status code:304
2. request header:

	![image](https://echizen.github.io/assets/blog-img/blog2016030502.jpg)
  
3. 特点：`cache-control:max-age=0`、`if-modify-since`

### cmd+shift+R(windows为ctrl+f5)

1. status code:200
2. request header:

	![image](https://echizen.github.io/assets/blog-img/blog2016030503.jpg)
   
3. 特点：`Cache-Control:no-cache`、`Pragma:no-cache`


# request headers中与缓存有关的字段

1. If-Modified-Since

	用于GET和HEAD请求，它的值是文档的最后改动时间，值来源于客户端当前时间或者该资源上次被加载的时间。客户端可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个返回资源的条件，只有改动时间迟于指定时间时服务器才会返回资源，否则会返回一个304（Not Modified）状态。当`If-None-Match`存在时，改参数失效。
   

2. Cache-Control
	
	值可以为`max-age=秒数`或`no-cache`，max-age定义文件在指定的时间内无需去服务端检查是否有更新，单位是秒；no-cache指定浏览器每次都要去服务端检查文件是否有更新。
	
	max-age=0和no-cache区别：max-age系列是由浏览器判断的，如果缓存时间超过设定的秒数就去服务器取，max-age=0和If-Modified-Since共同使用后还会出现不从服务器重新获取文件的情况，因为If-Modified-Since告诉浏览器文件还未过期。而no-cache告诉浏览器一律不走缓存，一定会去取服务器端数据。
	
3. Pragma

   HTTP/1.0 caches协议没有实现Cache-Control（在http1.1实现）， 实现了 Pragma: no-cache的功能。 请求头中如果包含pragma头的时候，服务器端会将该请求转发给源服务器。 指定“no-cache”值表示服务器必须返回一个刷新后的文档，即使它是代理服务器而且已经有了页面的本地拷贝。

4. If-None-Match

	用于GET和HEAD请求，对于拥有标签标记的资源，值为标签内容在这个标签未被修改的情况下获取缓存，返回304（ 304 Not Modified ）。
	
# response headers中对应的缓存相关字段
	
	
1. Last-Modified

	与请求头中的If-Modified-Since配合使用。
	
	在浏览器第一次请求某一个URL时，服务器端的返回状态会是200，内容是你请求的资源，同时有一个Last-Modified的属性标记(HttpReponse Header)此文件在服务期端最后被修改的时间。


	客户端第二次请求此URL时，根据HTTP协议的规定，浏览器会向服务器传送If-Modified-Since报头(HttpRequest Header)，询问该时间之后文件是否有被修改过。如果服务器端的资源没有变化，则自动返回HTTP304（NotChanged.）状态码，内容为空，这样就节省了传输数据量。当服务器端代码发生改变或者重启服务器时，则重新发出资源，返回和第一次请求时类似。从而保证不向客户端重复发出资源，也保证当服务器有变化时，客户端能够得到最新的资源。
	
2. Expires
	
	与请求头中Cache-Control配合使用。HTTP 1.0 版本，在给出的日期/时间后，资源的响应被认为是过时。需和Last-Modified结合使用。

	
3. Etag

	与请求头中的If-None-Match配合使用。
	
	HTTP协议规格说明定义ETag为“被请求变量的实体标记”。简单点即服务器响应时给请求URL标记，并在HTTP响应头中将其传送到客户端，类似服务器端返回的格式。客户端的使用If-None-Match查询更新。
如果ETag没改变，则返回状态304。

4. Cache-Control	

	通知从服务器到客户端内的所有缓存机制，表示它们是否可以缓存这个对象及缓存有效时间。其单位为秒

# 各缓存相关设置优先级

### Last-Modified/ETag 与 Cache-Control/Expires

配置 Last-Modified/ETag 的情况下，浏览器再次访问统一 URI 的资源，还是会发送请求到服务器询问文件是否已经修改，如果没有，服务器会只返回影响吗 304 （图第二张图所示）回给浏览器，代表资源没有变化，告诉浏览器直接读取缓存数据

Cache-Control/Expires 不同，如果检测到本地的缓存还是有效的时间范围内，浏览器直接使用本地副本，不会发送任何请求。两者一起使用时，**Cache-Control/Expires的优先级要高于Last-Modified/ETag**。即当本地副本根据Cache-Control/Expires发现还在有效期内时，则不会再次发送请求去服务器询问修改时间（Last-Modified）或实体标识（Etag）了。

### Cache-Control 与 Expires

Cache-Control 与 Expires 作用一致，都是指当前资源的有效期，控制浏览器是否直接从浏览器获取数据还是重新发送请求到服务器取数据。Cache-Control 的选择更多，设置更细致，如果响应头中同时存在，**Cache-Control优先级高于 Expires**。

# 不会被缓存的请求

1. 常规post请求不能被缓存，get请求可以
2. HTTP 信息头中包含Cache-Control:no-cache，pragma:no-cache，或Cache-Control:max-age=0 等告诉浏览器不用缓存的请求
3. 需要根据Cookie，认证信息等决定输入内容的动态请求
4. 经过HTTPS安全加密的请求
5. HTTP 响应头中不包含 Last-Modified/Etag，也不包含 Cache-Control/Expires 的请求

# 黄金外链

[https://tools.ietf.org/html/rfc7232#section-3](https://tools.ietf.org/html/rfc7232#section-3)

[Increasing Application Performance with HTTP Cache Headers](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers)

[Caching Tutorial](https://www.mnot.net/cache_docs/)



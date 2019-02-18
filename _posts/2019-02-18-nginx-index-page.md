---
layout: post
title: "一个常见需求的nginx配置踩雷"
description: "nginx配置首页、静态资源代理、web服务代理"
category: tech
tags: ['全栈']
---
{% include JB/setup %}

最近想给做的node应用服务提供nginx静态资源代理层。之前没做是不要约束大家的路径，后来想想不想被约束的开发可以不用，但是作为基础服务这个能力还是要有的，还有想用的人呢。况且，nginx代理静态资源可比node性能高多了，特别是大文件。

本以为很快就能配完测好上线了，好歹我也还是比较熟nginx的，结果一动手，发现踩了一片雷。。。

需求非常常见：
1. 静态资源走nginx代理
2. web接口代理给node应用
3. 首页可以免路径，直接通过域名xxx.com访问

静态资源可以约束下路径到`/static`文件夹下。

## location 优先级

最折腾人的就是第3项。脑子一拍最先想到配置就是用location的`= /`：

```
server {
  listen 80;
  root /Users/echizen/demo/nginx-learn/site1;

  location = / {
    index /static/index.html;
  }

  location / {
    proxy_pass http://127.0.0.1:8020;
  }
}
```

但是测试时一直走到proxy_pass的第二个location block了。翻下[location配置文档](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)：

> using the “=” modifier it is possible to define an exact match of URI and location. If an exact match is found, the search terminates. For example, if a “/” request happens frequently, defining “location = /” will speed up the processing of these requests, as search terminates right after the first comparison. Such a location cannot obviously contain nested locations.

还给了个案例，看着优先级规则没理解错啊：

> location = / {
    [ configuration A ]
  }

  location / {
      [ configuration B ]
  }

  location /documents/ {
      [ configuration C ]
  }

  location ^~ /images/ {
      [ configuration D ]
  }

  location ~* \.(gif|jpg|jpeg)$ {
      [ configuration E ]
  }
  The “/” request will match configuration A, the “/index.html” request will match configuration B, the “/documents/document.html” request will match configuration C, the “/images/1.gif” request will match configuration D, and the “/documents/1.jpg” request will match configuration E.


## index

毕竟代码量少，挨个怀疑就怀疑到`index`了，一翻[index配置文档](https://nginx.org/en/docs/http/ngx_http_index_module.html#index)，果然踩雷，连例子都甩出来了：

> It should be noted that using an index file causes an internal redirect, and the request can be processed in a different location. For example, with the following configuration:

  location = / {
      index index.html;
  }

  location / {
      ...
  }
  a “/” request will actually be processed in the second location as “/index.html”.

index内部是通过重定向实现的啊，就给重定向到`location /`了


## try_files

那就用try_files吧，不过一开始是拒绝这个玩意的，因为我明明知道准确路径，为啥要try_files来消耗资源，而且这货还必须至少2条file规则。后来仔细反思，其实挺好，这样多配几个规则可见兼容更多的index位置场景，把最常见的放第一个就好。

[try_files文档](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)

值的注意的是：

1. try_files最好在最后一条配上404，否则资源找不到nginx会陷入循环寻找导致500
2. 这货默认内部有最后一条规则走$uri，也就是路由的路径去寻找，所以看到没有命中自己index规则而是走了真正的路由路径也不要奇怪。
> If none of the files were found, an internal redirect to the uri specified in the last parameter is made

## 合度代理

之前孤陋寡闻，为了能将所有的路径都转发给node服务，规则配置的是：

```
location / {
  proxy_pass http://127.0.0.1:8020;
}
```

但是其实nginx对静态资源的代理性能更好，为啥不优先走nginx呢。所以可以优化成：

```
location / {
  try_files $uri @proxy;
}

location @proxy {
  proxy_pass http://127.0.0.1:8020;
}
```

## 安全性

目前为止，看上去需求都完成了：

```
server {
  listen 80;
  root /Users/echizen/demo/nginx-learn/site1;

  location = / {
    try_files index.html /static/index.html;
  }

  location / {
    try_files $uri @proxy;
  }

  location @proxy {
    proxy_pass http://127.0.0.1:8020;
  }
}
```

但是内心惴惴不安，这么合适么，`try_files $uri`让所有请求都先按请求路径查询了一遍，也就是说服务器根目录下的任何文件都被搞成静态资源代理了，服务器所有代码文件现在都可以在浏览器上直接访问查看了。。。虽说我是给内网应用配模板的，安全风险可控，但是总觉得是掉节操。

所以我不再所有代理所有路径了，只让static成为可访问路径：

```
server {
  listen 80;
  root /Users/echizen/demo/nginx-learn/site1;

  location = / {
    try_files /static/index.html index.html $uri=404;
  }

  location /static/ {
  }

  location / {
    proxy_pass http://127.0.0.1:8020;
  }
}
```

## 去除路径中的固定字段

node代理都是用`koa-static`习惯了，`/static/demo.js`只需要`http://xxx.com/demo.js`就能访问，无需非得在路由里加上`static`，虽说也可以用封装一下自定义版的`koa-static`，也借用`send`包但是把`static`路径补上，但是这样就改动了已有应用的路由要求，而且nginx明显有能力将`static`层去掉。

`try_files /static/$uri`搞起来就好。

```
server {
  listen 80;
  root /Users/echizen/demo/nginx-learn/site1;

  location = / {
    try_files /static/index.html index.html $uri=404;
  }

  location / {
    try_files /static/$uri @proxy;
  }

  location @proxy {
    proxy_pass http://127.0.0.1:8020;
  }
}
```

不过这也不是最优方案，因为所有请求都会先去匹配`/static/$uri`, 性能再高也是要去判断文件是否存在的，是个消耗。只是对于我的场景，内网应用，保持一致的习惯接入用户最小的改动比一点点的性能损耗来说是合适的。

可见最终这个nginx层对开发来说无感知，只是要求如果想使用nginx的能力就把静态资源放站点的`/static`路径就好，如果不使用这个不走nginx配置指定的首页，请求也会走到node服务的接口。

## 残留的疑惑

在此途中也遇到了2个查了文档搜了google也无法解释的现象：

1. 如果把`try_files`的file顺序调一下，`index.html`放前面(该文件不存在)，`http:xxx.com`就不能正常代理到`static/index.html`了，而是走了`proxy_pass`。即：

  ```
  server {
    listen 80;
    root /Users/echizen/demo/nginx-learn/site1;

    location = / {
      try_files index.html /static/index.html;
    }

    location / {
      proxy_pass http://127.0.0.1:8020;
    }
  }
  ```
  难道是`try_files`是个获取文件的尝试，会按优先级命中所有路由？即1.获取/Users/echizen/demo/nginx-learn/site1/index.html静态文件，不存在。 2.尝试http://127.0.0.1:8020 代理，如果404的话继续下去 3.获取/Users/echizen/demo/nginx-learn/site1/static/index.html...

2.上面的写法不ok，但是加上`location /static/`的规则就正常工作了，这个强行我也解释不了了。。。即：

  ```
  server {
    listen 80;
    root /Users/echizen/demo/nginx-learn/site1;

    location = / {
      try_files index.html /static/index.html;
    }

    location /static/ {
    }

    location / {
      proxy_pass http://127.0.0.1:8020;
    }
  }
  ```

  难道是`try_files`只是管理路由优先级，并不做可访问性代理？所以一定要后面接的path路径本身在其他规则下可访问？。但是try_files的文档介绍它的功能是`Checks the existence of files in the specified order and uses the first found file for request processing`，明显是查找文件是否存在，存在就响应啊。


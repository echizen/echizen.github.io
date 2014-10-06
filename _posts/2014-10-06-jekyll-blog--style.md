---
layout: post
title: "jekyll blog 模板修改 创建自己的style"
description: "mac环境下搭建jekyll blog，创建自定义风格模板，并部署到gitub上"
category: tech
tags: []
---
{% include JB/setup %}
#mac环境下搭建github - jekyll blog，创建自定义风格模板

##前篇-博客搭建
------在头回家探亲的国庆佳节，我终于可以有时间搭建好了自己的blog，普天同庆！
先看官网教程搭建吧，如果喜欢折腾，搭个jekyll框架就好了，再自己折腾样式，如果秉着重视内容，就把jekyll-bootstrapclone下来，直接用他的默认样式.


[jekyll官网：http://jekyllrb.com/docs/installation/](http://jekyllrb.com/docs/installation/)

[jekyll中文站：http://jekyllcn.com/docs/installation/](http://jekyllcn.com/docs/installation/)

[jekyll-bootstrap 官网：http://jekyllbootstrap.com/usage/jekyll-quick-start.html](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)（我基本是按这个教程安装的，虽然后来把它的模板全撤了，换成自己的了，这篇教程将github与jekyll博客搭建流程写的很清晰）

     

如果不出意外，你都会遇到一些问题，遇到后就去google吧，基本都能找到答案。我就遇到了一些问题。

1、其实博客搭建的时间比较早，期间本地博客一直搭不上，`gem install jekyll`一直在报错，后来静候mac更新了Ruby、下载了RubyGems·······不是很了解这些语言，也不是很了解ios系统，所以这里也没什么可分享的。就是要看教程、出问题google，


2、按教程来的话，你敲`git push origin master`，一定会报错：

    $ git push origin master
    Permission denied (publickey).
    fatal: The remote end hung up unexpectedly

这时，参考[https://help.github.com/articles/generating-ssh-keys/](https://help.github.com/articles/generating-ssh-keys/) 

---


##主篇-风格主题搭建
见我的下一篇博客吧

---


##尾篇-发布博客
有两种方式发布内容：  

###1、`rake post title="Hello World"`

一般写博客都是用这句，这句代码会在 `_post\` 文件夹下新建一个文件，文件名由 _config.yml 里的`permalink: /:categories/:year-:month-:day-:title `的配置决定，都说文件名不要取中文，jekyll系统会把文件名中的中文过滤掉，由于文章中会有title的设置，所以不用中文命名也不是什么痛苦的事啦。

>`permalink:`的配置不是随心所欲的，目前只支持三种配置：  
![image](http://)

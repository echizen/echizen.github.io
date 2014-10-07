---
layout: post
title: "custome blog style"
description: "自定义jekyll blog 样式，自定义jekyll 博客模板，配置jekyll 博客，修改jekyll博客默认页面，"
category: tech
tags: [前端,css,样式,beginner,jekyll, blog, tutorial]
---
{% include JB/setup %}

#自定义jekyll博客主题

##1、文件结构与关联
![image](https://echizen.github.io/assets/blog-img/QQ20141006-2.png)

如图是clone下来的原始结构，其中，比较重要的几个是：

![image](https://echizen.github.io/assets/blog-img/QQ20141006-3.png)

###1.1 _includes:主要的样式文件，你所有的博文内容的模板页都在这儿。

 -**themes**:这里放的是模板文件。

--default.html:博文公共内容的模板文件。比如头和尾。

--page.html:对于`layout:page`的文件的模板文件。

--post.html:对于`layout:post`的文件的模板文件。

--settings.yml:主题的声明，一般内容是：

```
theme :
  name : echizen
```
对于静态文件引用时的`{{ASSET_PATH}}`变量很有效。该变量在`_config.yml`中定义，在`./_includes/JB/setup`中有一句`{% capture ASSET_PATH %}{{ BASE_PATH }}/assets/themes/{{ page.theme.name }}{% `而settings.yml中的定义就是给`page.theme.name`一个值，改值要和你主题文件夹名称相同。

 -**JB**:主要是系统配置的插件或模块的模板文件。
 
--comments-providers:评论模板，文件夹下还有disqus、facebook、intensedebate、livefyre四个文件，这个与_config.yml里的配置有关,如我的的这个配置使用的是disqus，那么disqus文件就生效,打开disqus文件可以看到这里其实是disqus官网上的一段嵌入的代码，他决定着评论的样式和功能，你要是喜欢折腾，也可以来这个文件动刀子。

```
comments :
    provider : disqus
```


--analytics-providers:博客数据分析插件的样式,作用同上。



###1.2  _layouts:


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

--anlytics、comments这2个文件是使用liquid语法调用上面提的相应文件,有点MVC的感觉，这个是controller,上面提的文件是view.

--剩下的几个文件，都是一些配置，变量的定义什么的，我基本没动他们，感兴趣的可以打开文件看看。文件的注释部分将其用法写的很清楚。不果setup可以看看，这里详细定义了`BASE_PATH`、`HOME_PATH`、`ASSET_PATH`,对写模板时静态文件的引入很有用。



###1.2  _layouts:
这里的3个文件`default.html`、`page.htm`l、`post.html`相当于contrllor用来调用view的，如default.html文件里，先是 YAML关于模板名的定义。然后再是liquid语言，先调用setup，来让页面中引用静态文件的变量能解析，然后是调用相应的模板。

```
---
theme :
  name : echizen
---
{% include JB/setup %}
{% include themes/echizen/default.html %}
```


###1.3 _plugins:插件

###1.4 _posts:发布的文章集。

###1.5 _sites:站点目录
别人能通过URL访问的文件都在这里，基本你不用操作本目录，你在其他目录下的操作，如果涉及到站点的访问，jekyll都会自动同步到该目录。

###1.6 assets:静态文件资源存放
css、js、img都可以存在这里。

###1.7 _config.yml:配置文件
这里有很多全局变量的配置，这里配置的变量，在整个站点都能访问，但是这里都是简单的变量声名定义，具体变量的功能性操作都在其他文件夹下具体定义，比如你尝尝会见到的灰常灰常重要的JB变量，具体的JB的内容操作都在_includes的文件夹下
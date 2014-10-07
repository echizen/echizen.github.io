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

>`permalink:`的配置中的`/`不是真的文件夹部署，只是url上的显示，其实如果是真的文件夹分布就完美了。目前支持三种默认的配置，可以简洁的用`permalink: date`这种形式配置。

![image](https://echizen.github.io/assets/blog-img/QQ20141006-1.png)

也就是说无论你是`permalink: /:categories/:year-:month-:day-:title `还是什么，都只能显示三种效果，
###2、修改头信息
打开新建的那个文件，新建的文件终端都会输出新建的文件路径，所以只用`open`命令就可以很轻松的用默认的编辑器打开该文件了。（Mac下是Mou打开markdown文件）

```
rake post title="jekyll blog 模板修改-创建自己的style"
Creating new post: ./_posts/2014-10-06-jekyll-blog--style.md
➜  echizen.github.io git:(master) ✗ open ./_posts/2014-10-06-jekyll-blog--style.md
```

新建的~~~XXXhello-world.md的头部有一段`YAML` 头信息，形如：

```
---
layout: post
title: "hello world"
description: ""
category: 
tags: []
---
```
layout:决定渲染的模板，post就是post.html啦，用`rake post`命令生成的默认都是post。

title:真正的会在归档等地方显示的页面标题。

description:为了SEO优化，方便搜索引擎搜到，最好写一点。

category:文章所在的分类，像我的博客是以category分页面展示的，这时候这个参数就非常重要。

tags:给文章打上标签，和category一样，这样两个都是要在页面模板上有处理时才方便。


###3、发布博客
这一步在我的电脑上不需要，我在本地手动新建的文件，不用`git add`、`git commit`就能`jekyll server`后在本地访问，除了修改`_config.yml`这种比较系统的文件需要`git push origin master`之外，其他的变动远程也会同步。。。这个情况我不太明白我clone的jekyll都是以什么规则自动同步的。养成一个好的习惯，建议大家每次有变动时还是同步一下。

如果是新增：`git add XXX -> git commit -m 'add XXX'`

如果是修改已存在的文件就： `git ci -am 'modify XXX'`

然后一致`git push origin master`。


ps:在中文下，markdown其实挺不好用的，不断切换英文的语法标签和中文。。。
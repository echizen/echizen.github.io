---
layout: post
title: "自定义jekyll博客主题"
description: "自定义jekyll blog 样式，自定义jekyll 博客模板，配置jekyll 博客，修改jekyll博客默认页面，"
category: tech
tags: [前端, css, jekyll, blog, tutorial]
---
{% include JB/setup %}

# 自定义jekyll博客主题

你要在别人的东西上动刀子，就必须要先了解别人的东西，随便改改，后遗症是很严重的。

# 1. YAML与liquid
jekyll静态博客系统之所以能这么智能，首先归功于这两者，所以你得先了解他们。不需要能掌握，至少要能看懂已经写好的一些配置，能模仿着写一些头信息。

YAML是会经常用到的，这个标记语言没什么符号，所以空格要特别小心，他的缩进在语法上有特殊含义。



# 2. 文件结构与关联
![image](https://echizen.github.io/assets/blog-img/QQ20141006-2.png)

如图是clone下来的原始结构，其中，比较重要的几个是：

![image](https://echizen.github.io/assets/blog-img/QQ20141006-3.png)

## 2.1 _includes:主要的样式文件，你所有的博文内容的模板页都在这儿。

 -**themes**:这里放的是模板文件。

--default.html:博文公共内容的模板文件。比如头和尾。

--page.html:对于`layout:page`的文件的模板文件。

--post.html:对于`layout:post`的文件的模板文件。

--settings.yml:主题的声明，一般内容是：

		theme :
		  name : echizen
		  
对于静态文件引用时的`{{ASSET_PATH}}`变量很有效。该变量在`_config.yml`中定义，在`./_includes/JB/setup`中有一句`{% capture ASSET_PATH %}{{ BASE_PATH }}/assets/themes/{{ page.theme.name }}{% endcapture %} `而settings.yml中的定义就是给`page.theme.name`一个值，改值要和你主题文件夹名称相同。

 -**JB**:主要是系统配置的插件或模块的模板文件。
 
--comments-providers:评论模板，文件夹下还有disqus、facebook、intensedebate、livefyre四个文件，这个与_config.yml里的配置有关,如我的的这个配置使用的是disqus，那么disqus文件就生效,打开disqus文件可以看到这里其实是disqus官网上的一段嵌入的代码，他决定着评论的样式和功能，你要是喜欢折腾，也可以来这个文件动刀子。

		comments :
			provider : disqus


--analytics-providers:博客数据分析插件的样式,作用同上。

--anlytics、comments这2个文件是使用liquid语法调用上面提的相应文件,有点MVC的感觉，这个是controller,上面提的文件是view.

--剩下的几个文件，都是一些配置，变量的定义什么的，我基本没动他们，感兴趣的可以打开文件看看。文件的注释部分将其用法写的很清楚。不果setup可以看看，这里详细定义了`BASE_PATH`、`HOME_PATH`、`ASSET_PATH`,对写模板时静态文件的引入很有用。



## 2.2  _layouts:
这里的3个文件`default.html`、`page.htm`l、`post.html`相当于contrllor用来调用view的，如default.html文件里，先是 YAML关于模板名的定义。然后再是liquid语言，先调用setup，来让页面中引用静态文件的变量能解析，然后是调用相应的模板。(本来想上代码的，无奈会被解析掉。。。)


## 2.3 _plugins:插件

## 2.4 _posts:发布的文章集。

## 2.5 _sites:站点目录
别人能通过URL访问的文件都在这里，基本你不用操作本目录，你在其他目录下的操作，如果涉及到站点的访问，jekyll都会自动同步到该目录。

## 2.6 assets:静态文件资源存放
css、js、img都可以存在这里。

## 2.7 _config.yml:配置文件
这里有很多全局变量的配置，这里配置的变量，在整个站点都能访问，但是这里都是简单的变量声名定义，具体变量的功能性操作都在其他文件夹下具体定义，比如你尝尝会见到的灰常灰常重要的JB变量，具体的JB的内容操作都在`_includes/JB`文件夹下。

## 2.8 其他根目录文件
.html文件是直接访问的一级页面，如默认的`archive.html`、`categories.html`、`pages.html`、`tags.html`，自己写的放在博客里的一级页面也可以扔这里，通过`rake page ="about.md"`这种方式创建的页面也在这级上。

还有一部分介绍性文件，如`.md文件`。还有一个非常重要的文件`Rakefile`，建议如果不是很了解它就不要动他，否则会挂掉。


# 3.开启自己的模板时代
就按我自己写模板的过程来写吧。
先将上几幅我的博客的截图吧，我的博客其实只有3套样式的页面，主页、分类列表页、博文内容页。

![image](https://echizen.github.io/assets/blog-img/QQ20141009-1.png)

![image](https://echizen.github.io/assets/blog-img/QQ20141009-2.png)

![image](https://echizen.github.io/assets/blog-img/QQ20141009-3.png)



## 3.1 首页
首页是访问github page 的username.github.io直接跳转到的页面，首页的文件名一定要是index.html或index.md，因为jekyll的URL路由把域名指向了这里，而且index.md比index.html先加载，所以两者你只能取一个啦。首页保存在根目录下。
	首页是最最重要的面子工程啦。首页也要头信息，如果首页风格与其他页面都不同就不要加layout了，因为你不需要引用模板。为了引用静态文件，一定要加

		theme :
		   name : your theme name

然后别忘了`include JB/setup`，这都是为了在页面中使用`{{ASSET_PATH}}`的变量。

然后你就可以像写一个普通的html页面一样去尽情的绘制你的模板吧。

输出分类信息的代码，这样就可以把你的分类信息显示出来了。（为了不被jekyll解析，我将以下代码中liquid的tag符号去除了）

		 assign pages_list = site.pages 
		 assign group = 'navigation' 
		 include JB/pages_list 

>说的深入一点，site.pages指的是所有头信息中定义了layout:pages的页面，而`group = 'navigation'`这句是将头信息中含有`group: navigation`的页面筛选出来了。然后调用了`_includes/JB/pages_list`页面输出具体的html代码，所以你如果要修改nav列表的html可以去这里。

## 3.2 设置导航
国人习惯分类写文章，所谓的分类其实就是jekyll上的`category`参数。

我先在根目录下新建我的分类的一级页面，我分了4类:tech、think、life、show，命名的时候我是`a_tech.html`，加了前缀是为了在页面显示时排序，因为你输出这些分类的的代码其实是一个遍历，遍历的顺序是通常的字母排序。

我的导航页面内容很简单，因为我调用了模板，我把首页当view来写的，导航的这些页面则是当controller来写的（mvc的概念是php中的，不知其他语言是否有，大家是否能意会）。

	
		---
		layout: page
		title: Tech
		header: Posts By Tech
		group: navigation
		theme :
		   name : echizen
		---

这里很重要的一句是`group: navigation`，通过上面的介绍你应该已经知道，这是决定这个文件是导航文件的一个重要项。

然后调用`include JB/setup`，`assign posts_collate = site.categories.tech`,`include JB/posts_collate`。
`assign posts_collate = site.categories.tech`：这句非常重要，`site.categories.category_name`就是循环获取这个category_name下的博客文章。

## 3.3 导航页面样式
我是在`_includes/echizen/page.html`页面设置样式的，而调用这个page的页面是`_layouts/page.html`，这个页面的`layout:default`,所以我设置的时候导航页面和普通的博文页面的一部分是公用的，这无所谓，看个人所需。然后去_includes/JB/posts_collate这个文件修改样式。

## 3.4 具体页面样式
其实这部分是看前端怎么划分，组织页面间样式可以多样性，你甚至可以让每个分类的样式不同。

虽然我是个折腾的前端，但是我没折腾到每个页面都独具一格。我是首页一种样式，分类一种样式，博客一种样式。分享一下我的修改经验。首页完全独立，分类页面和博文内容有共用内容。

### default.html
前面已经介绍了，样式文件在`_includes/JB/themes/echizen`下，echizen是我设置的主题名。主要的样式文件是`default.html`,这里放置分类页面和博文页面的公有内容。为了观者导航方便，这两个页面的导航都是应该有的，还有公用的页尾信息。

加入导航条（为了不被jekyll解析，我将以下代码中liquid的tag符号去除了）：

		
		 assign pages_list = site.pages	  
		 assign group = 'navigation'	  
		 include JB/pages_list ]
		

作为一名尊重别人劳动成果的的程序猿，我没有将页面尾部`<footer>`和`</footer>`之间的感谢原框架作者的内容，^_^

### page.html
在我的博客主题里这其实就是分类页的模板，里面用`liquid`引用了`content`，然后由于我在分类页的原controller页面调用了`include JB/posts_collate`，所以我去这个页面把文章列表按时间线的样式排列了下来。

### post.html
博文内容样式页，这里有一些文章标题、内容框架、标签样式、分页的样式。然后具体的pre、h3、p等这些样式可以写成css样式表，扔到assets/themes/theme_name/css下。

**注意default.html可以解析YAML，但是page.html和post.html不能解析YAML**

## 3.5 添加comment
   先从 Disqus, Intense Debate, livefyre,  Facebook Comments中选一个，去官网注册个帐号，我选的是[Disqus](https://disqus.com/)，在[这里](https://disqus.com/admin/create/)注册个号，然后将你注册的帐号名添加到_config.yml中
   
	   comments :
	    provider : disqus
	    disqus :
	      short_name : your_name




# 4.番外篇 
有看到 [大神](http://pizn.github.io/) 把jekyll默认的结构改的渣都不剩的，我也clone了他的代码看了作用,[https://github.com/pizn/pizn.github.com](https://github.com/pizn/pizn.github.com)，感谢他的代码的启发作用。无奈功底不深，无法改成这幅样子，我更多的参考的还是默认模板，所以我的主题是在默认模板下修改的。但是操作过一遍后我发现其实不改其原框架结构，按其原则也能达到随心所欲的页面的效果，再结合liquid和YAML，足够折腾了，所以不是很了解其内部关系时，我建议大家也按其规则修改。

## 关于markdown解析
原来markdown具有多种渲染器，这些渲染器对markdown语法的解析还不一样。jekyll使用的是 Redcarpet 的markdown渲染器，他对代码块的```不支持，代码块必须是缩进4个空格，好不方便，不过用快捷键解决的话也能接受。


# 5.主要解决的问题
###  jekyll server 本地blog运行时,mac会报错一群cant'find ··· in atom.xml的错误。
解决：将_config.yml中的`pygments: true `替换为`highlighter: pygments`.

###  `{{ASEET_PATH}}`只能在_includes下的defalut.html中使用，而不能在post.html、page.html中使用。
解决：在`_layout`下的post.html和page.html中加入头信息：

		theme :
		   name : theme_name
		   
并将`Rakefile`中137至145行的内容注释掉，替换成（为了不被jekyll解析，我将以下代码中liquid的tag符号去除了）：

		# if File.basename(filename, ".html").downcase == "default"
        #   page.puts "---"
        #   page.puts File.read(settings_file) if File.exist?(settings_file)
        #   page.puts "---"
        # else
        #   page.puts "---"
        #   page.puts "layout: default"
        #   page.puts "---"
        # end 
        page.puts "---"
        page.puts File.read(settings_file) if File.exist?(settings_file)
        page.puts "layout: default" unless File.basename(filename, ".html").downcase == "default"
        page.puts "---"
        page.puts "% include JB/setup %"
        page.puts "% include themes/#{theme_name}/#{File.basename(filename)} %"
        
        


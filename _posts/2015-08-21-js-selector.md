---
layout: post
title: "从jquery源码-sizzle css选择器引擎看兼容的js选择器"
description: "jquery源码、sizzle源码之选择器部分研究，在IE9以下替代getElementsByClassName的方法。看源码的建议"
category: tech
tags: [js,pro,jquery]
---
{% include JB/setup %}

为什么我会研究选择器？因为之间面试遇到一个问题：我们知道，getElementsByClassName的兼容性不好，你来写一个函数，使其拥有getElementsByClassName相同的功能，你会怎么写？

。。。由于个人经历，以前写php的，大二去了创业公司写前端，从此夜以继日，用的都是封好的接口，我对原生js操作dom这块非常非常的不熟，首先我不知道getElementsByClassName的兼容性不好。。。其次我以为他只能作用在document上。。。所以这个问题，兼容性不好的替代方案，我还真答不上来。

面完后，我想这个问题应该不简单，想到jquery选择器兼容性那么好，于是想去看jquery源码。

getElementsByClassName在IE9及以上才可用。

## 看源码的建议

本人首次看这么大的框架源码，一路踩坑，看了2天才把选择器部分勉强看完。。。总结几条看源码的建议：

- 拆分源码，带着目的性的看。不然源码那么长，没有清晰的目的，你就像走入了浩瀚的宇宙。之前就想有时间看看源码级的东西，可是一打开大一点的框架，看着几万行的代码，整个人就不好了。这次因为是好奇jquery的选择器，目的性非常强，就是想看选择器部分，所以看的还是津津有味。

- 拆分模块后，只研究该模块下有关的函数，不要每次看到一个函数都跟踪进去看源码，这样一环套一环，你一个模块估计就要看一个框架的代码量了，对于辅助性的函数，把它当做已知就好，去官网看看api，这个函数是什么总用，参数是什么，来哦接这些就好，抓住重点。

- 调试模式开起来啊。一定要借助chrome强大的断点和逐行跟踪能力。说来惭愧，第一天我真的不靠工具靠眼力看，那么多逻辑判断，文件切换，看的我都快吐了也没明白逻辑。后来下了个为压缩的jquery.js，在浏览器下一行行跟着源码跑，才搞懂逻辑。有什么不明白的地方，在去模块中的源代码里慢慢钻研。

- 这一条应该都知道吧，源码要看开发版的，不会有人傻的看发行版的一个jquey.js文件吧。。。jquery已经引用了requirejs，模块划分非常清晰。

![image](https://echizen.github.io/assets/blog-img/QQ20150821-1@2x.png)

## jquery 选择器设计



jquery选择器的核心是sizzle CSS选择器引擎。

	jQuery.find = Sizzle;
	
所有调用find方法的其实都是用了Sizzle选择器引擎。当然简单的选择id是会直接调用getElementById接口，而不用这么麻烦的。

sizzle和jquery是同一个作者，sizzle号称是最高效的css选择器引擎。精华我觉得是正则匹配。。。令我头疼的各种正则表达式。

我并没有真的读懂每一个细节，也不想叙述细节，关于jquery源码的研究网上一抓一大把。我这里只讲一下他的实现思路。也不讲复杂组合的选择器，先将最简单的单个选择器，形如：#id，.class这类。

### 拆分选择对象

sizzle根据正则表达式划分出选择器的类型，进行词法分析，对于复杂的组合型选择器，会拆分成数组，一个个的元素匹配。随便贴一个属性的选择器感受一下他的强大。

	// http://www.w3.org/TR/css3-selectors/#whitespace
	whitespace = "[\\x20\\t\\r\\n\\f]",

	// http://www.w3.org/TR/CSS21/syndata.html#value-def-identifier
	identifier = "(?:\\\\.|[\\w-]|[^\\x00-\\xa0])+",

	// Attribute selectors: http://www.w3.org/TR/selectors/#attribute-selectors
	attributes = "\\[" + whitespace + "*(" + identifier + ")(?:" + whitespace +
		// Operator (capture 2)
		"*([*^$|!~]?=)" + whitespace +
		// "Attribute values must be CSS identifiers [capture 5] or strings [capture 3 or capture 4]"
		"*(?:'((?:\\\\.|[^\\\\'])*)'|\"((?:\\\\.|[^\\\\\"])*)\"|(" + identifier + "))|)" + whitespace +
		"*\\]",

### id、tag

getElementById、getElementsByTagName

### class

1. 支持getElementsByClassName的浏览器使用getElementsByClassName（IE9及以上及现代浏览器）
2. 如果不支持getElementsByClassName的浏览器，支持querySelectorAll的浏览器使用querySelectorAll(IE8以上及现代浏览器)
3. 如果不支持getElementsByClassName和querySelectorAll，那就先通过context.getElementsByTagName(*)选择出选区所有节点（context是你要搜索的element范围区），然后通过getAttribute("class")来取得class值，匹配筛选，getAttribute兼容性非常好，IE6以上和所有主流浏览器，基本不存在兼容性问题了。具体代码有这么一段：

		"CLASS": function( className ) {
            var pattern = classCache[ className + " " ];

            return pattern ||
                (pattern = new RegExp( "(^|" + whitespace + ")" + className + "(" + whitespace + "|$)" )) &&
                classCache( className, function( elem ) {
                    return pattern.test( typeof elem.className === "string" && elem.className || typeof elem.getAttribute !== "undefined" && elem.getAttribute("class") || "" );
                });
        }


至此，大的思路介绍完毕，至于细节，各种情况的考虑等等，得真的拜读源码才能感受到作为一款框架的严谨程度之可怕。

## sizzle组成

Sizzle的整体结构如下：

（1）Sizzle主函数，里面包含选择符的切割，内部循环调用主查找函数，主过滤函数，最后是去重过滤。

（2）其他辅助函数，如 uniqueSort, matches ,matchesSelector。

（3）Sizzle.find主查找函数

（4）Sizzle.filter主过滤函数

（5）Sizzle.selectors 包含各种匹配用的正则，过滤用的正则，分解用的正则，预处理函数，过滤函数。

（6）根据浏览器的特征设计makeArray，sortOrder，contains等方法。

（7）根据浏览器的特征重写Sizzle.selectors中的部分查找函数，过滤函数，查找次序。

（8）若浏览器支持querySelectorAll,那么用它重写Sizzle，将原来的Sizzle作为后备方案包裹在新的Sizzle里面。

（9）其他辅助函数，如：isXML,posProcess。


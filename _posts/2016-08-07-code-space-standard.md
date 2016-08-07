---
layout: post
title: "代码规范-从空格到代码格式化"
description: "代码规范-关于空格的思考，空格是否必要，如何愉快空格"
category: tech
tags: []
---
{% include JB/setup %}

最近被大大吐槽空格键坏了。。。我的代码里空格时有时无，一般都是加上的（手动加的。。。），写的high了，为了追求速度，也会某些片段忽略了。。。

我就来扒扒空格。其实是安利代码格式化工具。

# 代码格式规范

先谈谈我对代码格式规范的理解，代码格式规范还真的没有一个统一的标准，每家公司的具体细则可能都不一样，但是一个团队内部的人应该一样，这样互相阅读代码时才能有最佳体验。

### 具体的格式规范是有争议的
譬如tab=2 space or4 space就让人说不出来到底哪个更优，来个讨论都能让双方打起来。

### 规范条例的初衷
- 为了减少错误。如在函数开头声明变量，防止内部变量声明提升带来的bug
- 方便阅读。换行，空格都有这个原因。
- 为了浏览器等解析更高效。譬如避免定义css原子类。
- 为了高效。譬如js语句结尾不加分号

规范不是统一的，一些矛盾的规范条例是分不出来谁好谁差，只要满足上面这些优点都是有益的，但有时候可能会满足其中一条又违背另一条。譬如空格的存在方便了阅读，但是如果手动加就影响了效率。一个团队遵从同一规范就好，对于有争议的条款规范的制定可能会遵从团队大多数人的习惯风格。


# 哪里该有空格（常见规范中）

## js

- if / else / for / while / function / switch / do / try / catch / finally 关键字后。如`if (condition) { ... }`
- 函数声明名称与参数之间。如`function name (arg) { ... }`(此条有争议)
- 二元运算符两侧。如`var a = !arr.length;`
- 代码块起始的左花括号 { 前。如

		if (condition) {
		}
		
		while (condition) {
		}
		
		function funcName() {
		}
		
- 在对象创建时，属性中的 : 之后

		var obj = {
		    a: 1,
		    b: 2,
		    c: 3
		};

- 不位于行尾的`,` 和 `;` 后。`let a, b, c;`
- generator的`*`之后

		function* foo() {
		}
		
		const foo = function* () {
		}
	
## css

- 选择器 与 `{` 之间

		.selector {
		}
	
- 属性名 后的 `:` 与 属性值 之间。`margin: 0;`

- 列表型属性值 书写在单行时，`,` 后。`font-family: Arial, sans-serif;`

		
#  工具的力量

我觉得如果每一个该有空格的地方都自己手动加空格，效率真的有点低。所以我去求助工具了。不仅可以帮你在该加空格的地方加空格，还可以一起把规范中其他关于格式的条例的格式化做掉。

## sublime plugin - HTML-CSS-JS Prettify

### 功能

此插件可以格式化html、css、js，可以抛弃jsFormat了。基于node，所以要有node环境，安装node。

### 使用

- 默认shift+cmd+H快捷键调起对当前窗口文件的格式化。
- 右键选择HTML/CSS/JS prettify -> prettify code。

- 还可以配置保存时即格式化，preferences -> package setting -> HTML/CSS/JS prettify -> set plugin options。HTMLPrettify.sublime-settings文件里`format_on_save:true`。
- 也可以为每个库配置独特的格式化文件，在库的根目录下建`.jsprettifyrc`，配置规则，他的优先级比全局设置的`.jsprettifyrc`优先级更高。

### 规则

shift+cmd+p -> 输入prettify -> 选择HTMLPretiffy:set prettify preference(或者preferences -> package setting -> HTML/CSS/JS prettify -> set Prettify perferences）,`.jsprettifyrc`里配置规则，默认规则是：
	
	  // Details: https://github.com/victorporof/Sublime-HTMLPrettify#using-your-own-jsbeautifyrc-options
	  // Documentation: https://github.com/einars/js-beautify/
	  "html": {
	    "allowed_file_extensions": ["htm", "html", "xhtml", "shtml", "xml", "svg"],
	    "brace_style": "collapse", // [collapse|expand|end-expand|none]Put braces on the same line as control statements (default), or put braces on own line (Allman / ANSI style), or just put end braces on own line, or attempt to keep them where they are（没理解。。。）
	    "end_with_newline": false, // 结尾空出一行
	    "indent_char": " ", // 缩进字符类型（空格或者tab）
	    "indent_handlebars": false, // e.g. {{#foo}}, {{/foo}}
	    "indent_inner_html": false, // 缩进 <head> 和 <body> 片段
	    "indent_scripts": "keep", // [keep|separate|normal]
	    "indent_size": 4, // 缩进单位（这里是4个空格的意思）
	    "max_preserve_newlines": 0, // 在一个chunk中允许的最大换行数（0表示不允许）
	    "preserve_newlines": true, // 元素前的换行是否被允许存在（仅仅对于elemnet起作用，对tags和text都不起作用）
	    "unformatted": ["a", "span", "img", "code", "pre", "sub", "sup", "em", "strong", "b", "i", "u", "strike", "big", "small", "pre", "h1", "h2", "h3", "h4", "h5", "h6"], // 不被format的标签（没懂为何有这个选项）
	    "wrap_line_length": 0 // Lines should wrap at next opportunity after this number of characters (0 disables)
	  },
	  "css": {
	    "allowed_file_extensions": ["css", "scss", "sass", "less"],
	    "end_with_newline": false, // 结尾空出一行
	    "indent_char": " ", // 缩进字符方式（空格:` `;tab:`\t`）
	    "indent_size": 4, // 缩进单位（这里是4个空格的意思）
	    "newline_between_rules": true, // 每一条css规则之间插入一个空行
	    "selector_separator": " ",
	    "selector_separator_newline": true // 是否用空行分割选择器 (e.g. "a,\nbr" or "a, br")
	  },
	  "js": {
	    "allowed_file_extensions": ["js", "json", "jshintrc", "jsbeautifyrc"],
	
	    // Set brace_style
	    //  collapse: (old default) 将`{`、`}`置于控制语句同一行
	    //  collapse-preserve-inline: (new default) 与collapse规则一直但是对es6 import结构支持`{`不分行。 https://github.com/victorporof/Sublime-HTMLPrettify/issues/231
	    //  expand: 将`{`、`}`置于新一行(Allman / ANSI style)
	    //  end-expand: 将`}` 置于新一行
	    //  none: 不去处理，与原位置一直
	    "brace_style": "collapse-preserve-inline",
	
	    "break_chained_methods": false, // 将链式调用的函数置于新一行
	    "e4x": false, // Pass E4X xml literals through untouched
	    "end_with_newline": false, //结尾空出一行
	    "indent_char": " ", // 缩进字符方式（空格:` `;tab:`\t`）
	    "indent_level": 0, // Initial indentation level
	    "indent_size": 4, // 缩进单位（这里是4个空格的意思）
	    "indent_with_tabs": false, // 使用tab缩进，会覆盖 `indent_size` 和`indent_char`的设置
	    "jslint_happy": false, // 是否强制使用`jslint-stricter`的模式
	    "keep_array_indentation": false, // 保留数组缩进
	    "keep_function_indentation": false, // 保留函数缩进
	    "max_preserve_newlines": 0, // 在一个chunk中允许的最大换行数（0表示不允许）
	    "preserve_newlines": true, // 是否保留换行
	    "space_after_anon_function": false, // 匿名函数function关键字和括号之间的空格是否需要, "function()" vs "function ()"
	    "space_before_conditional": true, // 是否添加条件语句关键词和括号之间的空格, "if(true)" vs "if (true)"
	    "space_in_empty_paren": false, // 函数空参数时是否添加padding值, "f()" vs "f( )"
	    "space_in_paren": false, // 是否在函数参数与括号之间插入空格, ie. f( a, b )
	    "unescape_strings": false, // Should printable characters in strings encoded in \xNN notation be unescaped, "example" vs "\x65\x78\x61\x6d\x70\x6c\x65"
	    "wrap_line_length": 0 // Lines should wrap at next opportunity after this number of characters (0 disables)
	  }
	}


### 不足之处

- `allowed_file_extensions`指定文件类型，发现不支持jsx，这就尴尬了，react的jsx文件都没法格式化，会将类似html格式的jsx按js语法格式化。
- css格式化有些Bug，设置了`newline_between_rules:true`，还是会删除规则之间的空行。我觉得有空行可读性更高。

## sublime plugin: Sublimelinter&&Sublimelinter-contrib-eslint

毕竟HTML-CSS-JS Prettify只是代码格式化工具，还有很多代码规范不是格式化就能解决的，这时候将上述HTML-CSS-JS Prettify代码格式化工具与eslint代码规范插件配合使用，利用eslint检查其他非格式的规范。

### 安装

	npm install eslint -g
	npm install babel-eslint -g
	
再通过package control安装Sublimelinter和Sublimelinter-contrib-eslint。

### 配置

修改Sublimelinter 配置，Preferences->Package Settings->SublimeLinter->Settings-User。

debug: true 开启 debug 模式 paths: 根据具体环境，设置 ESLint 路径，我的mac下是`/usr/local/bin`

	{
	    "user": {
	        "debug": true,
	        "linters": {
	            "eslint": {   
	                "excludes": ["*.html"]
	            }
	        },
	        "paths": {
	            "osx": [
	                "/usr/local/bin"
	            ]
	        }
	    }
	}

重启 Sublime Text。

### 规则配置

在项目根目录下新建` .eslintrc`文件配置。具体规则参考官网，非常详细。

[http://eslint.org/docs/rules/](http://eslint.org/docs/rules/)

然后保存时就能看到提示，不合格的行开头会有红点标识。

## standard 

号称"JavaScript Standard Style",能够替代eslinc和jshint，不需要配置文件。

但是他按照自己的规范检查你的文件，且他的规范可能和你团队指定的不同，所以使用它必须遵从他的规范。

[https://github.com/feross/standard](https://github.com/feross/standard)

# 参考

[https://github.com/fex-team/styleguide/blob/master/javascript.md](https://github.com/fex-team/styleguide/blob/master/javascript.md)

[http://airbnb.io/javascript/](http://airbnb.io/javascript/)

[https://github.com/victorporof/Sublime-HTMLPrettify#using-your-own-jsbeautifyrc-options](https://github.com/victorporof/Sublime-HTMLPrettify#using-your-own-jsbeautifyrc-options)

[https://www.npmjs.com/package/eslint-config-sm](https://www.npmjs.com/package/eslint-config-sm)

这里还有个14年sideeffect.kr做的github代码风格统计，可见大多数代码是没有遵从现在的主流规范的。。。

[http://sideeffect.kr/popularconvention#javascript](http://sideeffect.kr/popularconvention#javascript)
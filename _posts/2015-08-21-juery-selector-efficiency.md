---
layout: post
title: "从sizzle源码看选择器性能"
description: "CSS选择器从右向左选择原因，原生js操作dom几种方法的性能，jquery选择器如何高效利用"
category: tech
tags: [js,jquery,pro]
---
{% include JB/setup %}

本来只是好奇getElementsByClassName兼容性解决方案，结果又是一脚踩了深水泥潭，一路看到了性能问题。

以我的经验和见识还没有资格谈性能问题，以下内容多是从网上看到的大牛们的分析，结合了自己的见解和对源码执行顺序的跟踪对这些观点的验证，更是一份笔记。

所谓jquery选择器，其实大多数是css选择器的包装。

##html渲染解析过程

浏览器从下载文档到显示页面的过程是个复杂的过程，这里包含了重绘和重排。各家浏览器引擎的工作原理略有差别，但也有一定规则。

- 简单讲，通常在文档初次加载时，浏览器引擎会解析HTML文档来构建DOM树，之后根据DOM元素的几何属性构建一棵用于渲染的树。渲染树的每个节点都有大小和边距等属性，类似于盒子模型（由于隐藏元素不需要显示，渲染树中并不包含DOM树中隐藏的元素）。

- 当渲染树构建完成后，浏览器就可以将元素放置到正确的位置了，再根据渲染树节点的样式属性绘制出页面。由于浏览器的流布局，对渲染树的计算通常只需要遍历一次就可以完成

## 为什么排版引擎解析 CSS 选择器时一定要从右往左解析

- HTML 经过解析生成 DOM Tree（这个我们比较熟悉）；而在 CSS 解析完毕后，需要将解析的结果与 DOM Tree 的内容一起进行分析建立一棵 Render Tree，最终用来进行绘图。Render Tree 中的元素（WebKit 中称为「renderers」，Firefox 下为「frames」）与 DOM 元素相对应，但非一一对应：一个 DOM 元素可能会对应多个 renderer，如文本折行后，不同的「行」会成为 render tree 种不同的 renderer。也有的 DOM 元素被 Render Tree 完全无视，比如 display:none 的元素。


- 在建立 Render Tree 时（WebKit 中的「Attachment」过程），浏览器就要为每个 DOM Tree 中的元素根据 CSS 的解析结果（Style Rules）来确定生成怎样的 renderer。对于每个 DOM 元素，必须在所有 Style Rules 中找到符合的 selector 并将对应的规则进行合并。选择器的「解析」实际是在这里执行的，在遍历 DOM Tree 时，从 Style Rules 中去寻找对应的 selector。

- 因为所有样式规则可能数量很大，而且绝大多数不会匹配到当前的 DOM 元素（因为数量很大所以一般会建立规则索引树），所以有一个快速的方法来判断「这个 selector 不匹配当前元素」就是极其重要的。

- 如果正向解析，例如「div div p em」，我们首先就要检查当前元素到 html 的整条路径，找到最上层的 div，再往下找，如果遇到不匹配就必须回到最上层那个 div，往下再去匹配选择器中的第一个 div，回溯若干次才能确定匹配与否，效率很低。

- 逆向匹配则不同，如果当前的 DOM 元素是 div，而不是 selector 最后的 em，那只要一步就能排除。只有在匹配时，才会不断向上找父节点进行验证。

- 但因为匹配的情况远远低于不匹配的情况，所以逆向匹配带来的优势是巨大的。同时我们也能够看出，在选择器结尾加上「*」就大大降低了这种优势，这也就是很多优化原则提到的尽量避免在选择器末尾添加通配符的原因。

我的理解： 按照标准css选择器效率的几个方面：

- 层级越少效率越高
- 下一层应该比上一层更精确

依照这个标准，可以推测css选择器算法的大概模型，也就知道上面的解困是对的。

css会先读最右边，左边的层级是为了验证右边的层并筛选。

按照css选择器规范，一个良好的css选择器可以长这样： `div .class`，而不应该是这样`.class div`。
第一种情况下，具体的.class元素节点少于div节点的概率远远大于相反情况。所以对于符合css规范的选择器，从右向左读显然更友好。

## sizzle的词法解析

所谓词法解析，就是将用户传进来的一个复杂的选择器`$(selector)`中的selector拆分成一个个简单的元素，并匹配类型，这样才能遍历寻找。

sizzle中有一个叫做tokenize的函数就是干这个的：

	tokenize = Sizzle.tokenize = function( selector, parseOnly ) {
		var matched, match, tokens, type,
			soFar, groups, preFilters,
			cached = tokenCache[ selector + " " ];
	
		if ( cached ) {
			return parseOnly ? 0 : cached.slice( 0 );
		}
	
		soFar = selector;
		groups = [];
		preFilters = Expr.preFilter;
	
		while ( soFar ) {
	
			// Comma and first run
			if ( !matched || (match = rcomma.exec( soFar )) ) {
				if ( match ) {
					// Don't consume trailing commas as valid
					soFar = soFar.slice( match[0].length ) || soFar;
				}
				groups.push( (tokens = []) );
			}
	
			matched = false;
	
			// Combinators
			if ( (match = rcombinators.exec( soFar )) ) {
				matched = match.shift();
				tokens.push({
					value: matched,
					// Cast descendant combinators to space
					type: match[0].replace( rtrim, " " )
				});
				soFar = soFar.slice( matched.length );
			}
	
			// Filters
			for ( type in Expr.filter ) {
				if ( (match = matchExpr[ type ].exec( soFar )) && (!preFilters[ type ] ||
					(match = preFilters[ type ]( match ))) ) {
					matched = match.shift();
					tokens.push({
						value: matched,
						type: type,
						matches: match
					});
					soFar = soFar.slice( matched.length );
				}
			}
	
			if ( !matched ) {
				break;
			}
		}
	
		// Return the length of the invalid excess
		// if we're just parsing
		// Otherwise, throw an error or return tokens
		return parseOnly ?
			soFar.length :
			soFar ?
				Sizzle.error( selector ) :
				// Cache the tokens
				tokenCache( selector, groups ).slice( 0 );
	};
	
Sizzle的词法解析后的Token格式如下 ：

	Token：{  
	   value:'匹配到的字符串', 
	   type:'对应的Token类型', 
	   matches:'正则匹配到的一个结构'
	}

type可能是：`ID,TAG,CLASS,ATTR,CHILD,PSEUDO`以及表示层级关系的`+,>, ,~`。

一个组合式的selector经过tokenize后会变成一个数组，每个值都是token格式的对象。

然后sizzle按照一定的顺序去匹配筛选这个数组中的元素，找到正确匹配的元素。


##sizzle匹配顺序（内容待更新）

当sizzle处理多层选择器，按照从右到左的原则，特殊选择符会从左到右。

分词操作之后并没有直接开始组装匹配方法，而是先做了一些find的操作。这里的find操作就可以对应到Expr里面的find，它执行的是查询操作，返回的是结果集。

可以这样理解，select利用“分词”得到的选择符根据它的type先将可以用find方法查找的结果集查出来。做find操作的时候，是按照选择符的顺序从左到右缩小结果集范围的。如果一个遍历下来，selector中的所有选择符都可以执行find操作，则直接将结果返回。否则，就进入前面介绍的“编译”执行过滤的流程了。

到这里，也可以顺过来，基本上理清楚Sizzle的工作流程了。前面留下的疑问到此时其实也不算疑问了，因为执行反向匹配过滤的时候，它的查找范围已经是经过层层过滤的最小集合了。而反向匹配过滤的方法对于它所对应的那些选择符，比如伪类之类的，其实也已经是一个高效的选择。

Sizzle.find主查找函数和Sizzle.filter过滤函数实现原理：

对js原生的4大查找函数，getElementById（针对id），getElementsByName（针对name），getElementsByTagName（针对标签名tagName，比如div,p），getElementsByClassName(针对class)，进行一层封装，浏览器支持的话，就返回数组或者NodeList，不支持的，就返回undefined。

这里需要讲一下种子集。

###种子集

种子集就是通过最右边的选择器组得到的元素集合。比如："div.aaa span.bbb"，最右边的选择器组就是"span.bbb"，这时引擎会根据浏览器的支持情况选择getElementsByTagName(span)或getElementsByClassName(bbb)得到一组元素，然后再通过class(bbb)或tagName(span)进行过滤，这时得到的集合就是种子集。

种子集是分两步筛选出来的，首先，通过Sizzle.find得到一个大体的结果，然后通过Sizzle.filter过滤。那我们是先取span，还是.bbb呢？这里有一个准则，要确保我们后面的映射集（当我们取得种子集后，会将种子集一份，这就是映射集）最小。

为了达到此目的，这里有一个优化，原生选择器的调用顺序被放在一个Sizzle.selectors.order的数组中，对于低版本浏览器，其顺序为id,name,tagName，对于支持getElementsByClassName的浏览器，顺序为**id,class,name,tagName**。因为id只返回一个元素，class与样式相关，不是每个元素都有这个类名的，name属性使用到的几率比较少，而tagName排除的元素比较少。

所以Sizzle.find就会根据Sizzle.selectors.order数组，依次调用正则，从最右的选择器中切下需要的部分，找到粗糙的节点集合。(针对"span.bbb"，id调用正则时，找不到，然后class，调用正则，找到.bbb，因此就调用getElementsByClassName(bbb)得到一组数据，最后通过Sizzle.filter过滤取到的数据，过滤条件是tagName(span))

###映射集

当我们取得种子集后，会将种子集一份，这就是映射集。

种子集是由一个选择器组选出来的，这时如果选择符不为空（前面是"div.aaa"），必然往左就是关系选择器（父亲，兄弟，后代），关系选择器会让引擎去选取其兄长或父亲，把这些元素置换到映射集对等的位置上（个数不变，因此映射集和种子集的数量总是相当）。然后到下一个选择器组时（"div.aaa"），就是过滤操作了。主过滤函数Sizzle.filter会调用Sizzle.selectors下的N个过滤函数对这些元素进行检测，将不符合的元素替换为false。因此到最后要去重排时，映射集是一个包含布尔值与元素节点的数组。

## sizzle高效原因

首先，从处理流程上，它总是先使用最高效的原生方法来做处理。前面一直在介绍的还只是Sizzle自身的选择器实现方法，真正Sizzle执行的时候，对于css选择器，它还会先判断当前浏览器是否支持querySelectorAll原生方法。如果支持的话，则优先选用此方法，浏览器原生支持的方法，效率肯定比Sizzle自己js写的方法要高，优先使用也能保证Sizzle更高的工作效率。（关于querySelectorAll可以上网查阅更多资料）。在不支持querySelectorAll方法的情况下，Sizzle也是优先判断是不是可以直接使用getElementById、getElementsByTag、getElementsByClassName等方法解决问题。

其次，相对复杂的情况，Sizzle总是选择先尽可能利用原生方法来查询选择来缩小待选范围，然后才会利用前面介绍的“编译原理”来对待选范围的元素逐个匹配筛选。进入到“编译”这个环节的工作流程有些复杂，效率相比前面的方法肯定会稍低一些，但Sizzle在努力尽量少用这些方法，同时也努力让给这些方法处理的结果集尽量小和简单，以便获得更高的效率。

再次，即便进入到这个“编译”的流程，Sizzle还做了我们前面为了优先解释清楚流程而暂时忽略、没有介绍的缓存机制。“编译”入口，会调用第三个核心方法superMatcher。跟踪进去看，compile方法将根据selector生成的匹配函数缓存起来了。还不止如此，再看tokenize方法，它其实也将根据selector做的分词结果缓存起来了。也就是说，当我们执行过一次Sizzle (selector)方法以后，下次再直接调用Sizzle (selector)方法，它内部最耗性能的“编译”过程不会再耗太多性能了，直接取之前缓存的方法就可以了。我在想所谓“编译”的最大好处之一可能也就是便于缓存，所谓“编译”在这里可能也就可以理解成是生成预处理的函数存储起来备用

在Sizzle中，当浏览器支持querySelectorAll方法时，会重写Sizzle。但是在重写时，会根据不同情况提出各种提速方案：

（1）getElementById还是比querySelectorAll速度快，因为getElementById只返回一个元素，而且内部做了缓存，但是querySelectorAll会返回拥有这个id值的多个元素，尽管页面id一般是唯一的，但如果出现了多个同样id的情况下，getElementById还是只返回一个元素，而querySelectorAll会返回多个。

（2）getElementsByTagName内部也使用了缓存，而且返回的是NodeList对象，querySelectorAll返回的是一个StaticNodeList对象，前面是动态的，后面是静态的。区别在于：document.getElementsByTagName("div") == document.getElementsByTagName("div"),返回真，document.querySelectorAll("div") == document.querySelectorAll("div")，返回false.返回true的，意味着它们拿到的同是cache引用。返回false意味着每次返回都是不一样的object。数据表明：创建一个动态的NodeList对象比创建一个静态的StaticNodeList对象快90%.

##使用选择器的几点建议

正确使用选择器引擎对于提高页面性能起了至关重要的作用。使用合适的选择器表达式可以提高性能、增强语义并简化逻辑。在传统用法中，最常用的简单选择器包括ID选择器、Class选择器和类型标签选择器。其中ID选择器是速度最快的，这主要是因为它使用JavaScript的内置函数getElementById()；其次是类型选择器，因为它使用JavaScript的内置函数getElementsByTag()；速度最慢的是Class选择器，其需要通过解析 HTML文档树，并且需要在浏览器内核外递归，这种递归遍历是无法被优化的。

Class选择器在文档中使用频率靠前，这无疑会增加系统的负担，因为每使用一次Class选择器，整个文档就会被解析一遍，并遍历每个节点。

对于`$("input[type=checkbox]")`和`$("input:checkbox")`,对于现代浏览器，`$("input[type=checkbox]")`可以使用`querySelectorAll`这种内置方来查找,$(‘input:text’)，采用了jQuery自定义的选择器表达式:text，.querySelectorAll()方法无法解析。所以前一种效率明显高于后一种。

###多用ID选择器 , 总是从#id选择器来继承

多用ID选择器，这是一个明智的选择。即使选择的元素没有ID，也可以从父级元素中添加一个ID选择器，这样就会缩短节点访问的路程。这是jQuery选择器的一条黄金法则。jQuery选择一个元素最快的方法就是用ID来选择了

###使一个选择器的右边更具有特征，相对而言，选择器的左边可以少一些特征性

由于css选择器引擎的从右到左解析的特性，右边约具有特征，下一步循环筛选的元素的基数越少，速度越快。

### 对于class选择器，尽可能使用"tag.class"类型的选择符

每使用一次Class选择器，如果不加tag,整个文档就会被解析一遍，并遍历每个节点，因为这时候sizzle需要用getElementsByTagName(*)，取出搜索域中的所有节点。但是加了tag后getElementsByTagName(tag)会大大缩小搜索范围。

### 避免过度的约束，让层级更简单

由于从右到左的解析原则，找到最右边seed子集后，左边的层级都用来筛选了，验证子集中哪些是符合的。层级越复杂，验证筛选的流程越多。

### 避免使用全局选择器*

无论你将*使用在哪一级，都会极大的降低性能，需要遍历每一层元素。

### 多用父子关系，少用嵌套关系

使用parent>child代替parent child。因为">"是child选择器，只从子节点里匹配，不递归。而" "是后代选择器，递归匹配所有子节点及子节点的子节点，即后代节点。

下面六个选择器，都是从父元素中选择子元素。你知道哪个速度最快，哪个速度最慢吗?

	$('.child', $parent)
	$parent.find('.child')
	$parent.children('.child')
	$('#parent > .child')
	$('#parent .child')
	$('.child', $('#parent'))
	
- $('.child', $parent)：给定一个DOM对象，然后从中选择一个子元素。jQuery会自动把这条语句转成$.parent.find('child')，这会导致一定的性能损失。

- $parent.find('.child')： 这条是最快的语句。.find()方法会调用浏览器的原生方法（getElementById，getElementByName，getElementByTagName等等），所以速度较快。

- parent.children('.child')：这条语句在jQuery内部，会使用$.sibling()和javascript的nextSibling()方法，一个个遍历节点。它比最快的形式大约慢50%

- $('#parent > .child')： jQuery内部使用Sizzle引擎，处理各种选择器。Sizzle引擎的选择顺序是从右到左，所以这条语句是先选.child，然后再一个个过滤出父元素#parent，这导致它比最快的形式大约慢70%。

- $('#parent .child')：这条语句与上一条是同样的情况。但是，上一条只选择直接的子元素，这一条可以于选择多级子元素，所以它的速度更慢，大概比最快的形式慢了77%。

- $('.child', $('#parent'))： jQuery内部会将这条语句转成$('#parent').find('.child')，比最快的形式慢了23%。

### 缓存jQuery对象

如果选出结果不发生变化的话，不妨缓存jQuery对象，这样就可以提高系统性能。养成缓存jQuery对象的习惯可以让你在不经意间就能够完成主要的性能优化。如：

	var a= $( ' .aaron' );   
	for (i = 0 ; i < 10000 ; i ++ ) ... {   
	     a.append(i);   
	}
	
### 多使用链式调用

如：

	$('#news').find('tr.alt').removeClass('alt').end().find('tbody').each(function() {
	        $(this).children(':visible').has('td').filter(':group(3)').addClass('alt');
	   });
   
链式调用优于性能的原因是链式将选择器选择结果缓存了。

采用find(),end(),children(),has,filter()等方法，来过滤结果集，减少$()查找方法调用，提升性能。

### 尽量使用符合CSS语法规范的CSS选择器表达式，以此来避免使用jQuery自定义的选择器表达式

CSS选择器表达式可以使用js内置方法querySelectorAll来查找，而jquery自定义选择器需要sizzle去遍历dom树根据修饰器过滤确定最后结果。

## 黄金外链

[jQuery 源码分析Sizzle引擎 - 词法解析](http://www.cnblogs.com/aaronjs/p/3300797.html)

[Sizzle引擎详解](http://www.lxway.com/8005986.htm)

[Sizzle的“编译原理”](http://tid.tenpay.com/?p=4007)



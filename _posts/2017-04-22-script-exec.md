---
layout: post
title: "动态插入的script脚本执行时间"
description: "动态插入的script脚本执行时间和顺序"
category: tech
tags: [js]
---
{% include JB/setup %}

在一些场景我们会动态插入script标签加载js。

譬如某个js文件不是很重要，并不是整个页面需要的脚本，可能只是某个功能需要的，这个功能可能是用户点击了某个按钮才触发，入口比较深。且和你页面本身的结构不同类，譬如你是基于react的页面，这个功能的js是jquery插件。这种js文件我一般采用动态加载方式引入。

如果你用js动态插入script，那么它什么时候执行呢，如果插入多个script，且之间有依赖关系，是否先插入的script先执行呢？

答案是： 不是

## demo案例

### 代码

`js-exec.js`:动态插入2个script到页面中，`test1.js`中定义了一个全局变量`obj`，`test2.js`加载完成后的`onload`事件中会去使用这个变量`obj`。`test1.js`和`test2.js`都在打印了信息方便查看执行顺序


    var getReadyForEditor = () => {
      console.log(obj.foo)
    }

    var editorJs = document.createElement("script")
    editorJs.src = "./test1.js"
    document.body.appendChild(editorJs)

    var editorJs2 = document.createElement("script")
    editorJs2.src = "./test2.js"
    editorJs2.onload = getReadyForEditor
    document.body.appendChild(editorJs2) 

`test1.js`: 控制台打印1，并且定义了`obj`变量

    console.log(1)
    var obj = {
      foo: 'foo'
    }
    
`test2.js`: 控制台打印2。

    console.log(2)
    
就是这么简洁的demo

### 执行

通过不断刷新，发现大概率是按照test1、test2的顺序执行，但是也有一部分是先执行test2再执行test1:

![http://s2.mogucdn.com/mlcdn/c45406/170420_8g0gk91ajad3b65lhcfibaj418dg4_1422x718.png](http://s2.mogucdn.com/mlcdn/c45406/170420_8g0gk91ajad3b65lhcfibaj418dg4_1422x718.png)

由截图可见，网络请求顺序是按照script插入的顺序，先插入到dom的先请求，但是请求时间不一样，test2比test1的请求时间短，内容先返回。看现象貌似结论是：**资源加载完成时执行，因此资源加载先完成的先执行**

## 猜测

我们都知道如果是非动态插入的script，是按照在html里出现的顺序执行的，但是现在动态插入的脚本，虽然先插入的script位于html的前面，后插入的在后面，但是执行顺序却没有按这个顺序来。

是不是因为浏览器不知道在一个script标签插入后还有没有下一个要插入，所以没法按顺序执行呢？那么我们一次性插入这2个标签会怎样？

    var getReadyForEditor = () => {
      console.log(obj.foo)
    }

    var editorJs = document.createElement("script")
    editorJs.src = "./test1.js"
    // document.body.appendChild(editorJs)

    var editorJs2 = document.createElement("script")
    editorJs2.src = "./test2.js"
    editorJs2.onload = getReadyForEditor
    // document.body.appendChild(editorJs2) 

    var docFrag = document.createDocumentFragment()
    docFrag.appendChild(editorJs) // Note that this does NOT go to the DOM
    docFrag.appendChild(editorJs2)
    document.body.appendChild(docFrag) 
    
通过`createDocumentFragment`创造文档片段，然后一次插入，这样浏览器该知道顺序了吧。

但是结果依旧没变。猜想错误！


## 文档

猜不出来，只能找理论支撑了。先去翻了W3C规范，但是抱歉，实在没看明白，找到想要的答案：

[https://www.w3.org/TR/html5/scripting-1.html#attr-script-src](https://www.w3.org/TR/html5/scripting-1.html#attr-script-src)

> If the element has a src attribute, does not have an async attribute, and does not have the "force-async" flag set
The element must be added to the end of the list of scripts that will execute in order as soon as possible associated with the Document of the script element at the time the prepare a script algorithm started.
    The task that the networking task source places on the task queue once the fetching algorithm has completed must run the following steps:
    If the element is not now the first element in the list of    scripts that will execute in order as soon as possible to which it was added above, then mark the element as ready but abort these steps without executing the script yet.
    Execution: Execute the script block corresponding to the first script element in this list of scripts that will execute in order as soon as possible.
    Remove the first element from this list of scripts that will execute in order as soon as possible.
    If this list of scripts that will execute in order as soon as possible is still not empty and the first entry has already been marked as ready, then jump back to the step labeled execution.
    
    
然后翻MDN的文档：

[https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)

有这么一段：

>  script-inserted scripts execute asynchronously in IE and WebKit, but synchronously in Opera and pre-4.0 Firefox. In Firefox 4.0, the async DOM property defaults to true for script-created scripts, so the default behavior matches the behavior of IE and WebKit. To request script-inserted external scripts be executed in the insertion order in browsers where the document.createElement("script").async evaluates to true (such as Firefox 4.0), set .async=false on the scripts you want to maintain order. 


## 真相

这就是我要找的！

原来是因为浏览器对动态插入的script标签，默认设置的是`async`。（各浏览器有区别）

我们知道`async`作用的js脚本时没有顺序的，异步加载，加载后执行。

因此特性，所以还有个`defer`，`defer`是异步加载，按script在文档中的顺序执行。

那我们的测试demo试一下，打印出来的`async`果真是`true`

![http://s2.mogucdn.com/mlcdn/c45406/170420_0bgkg16l2ljc3341444c96c32241h_632x188.png](http://s2.mogucdn.com/mlcdn/c45406/170420_0bgkg16l2ljc3341444c96c32241h_632x188.png)

## 如何让动态插入的script标签按插入顺序执行

既然问题出在async上，那么创建script标签时把他设置为false就好。

    var editorJs = document.createElement("script")
    editorJs.src = "./test1.js"
    editorJs.async = false
    document.body.appendChild(editorJs)

    var editorJs2 = document.createElement("script")
    editorJs2.src = "./test2.js"
    editorJs2.onload = getReadyForEditor
    editorJs2.async = false
    document.body.appendChild(editorJs2) 

再观察，即使test2比test1先加载完，也会等待test1执行完在执行了~

![http://s2.mogucdn.com/mlcdn/c45406/170420_262g6cce4kd6b6ajg60d152fl64f0_1440x690.png](http://s2.mogucdn.com/mlcdn/c45406/170420_262g6cce4kd6b6ajg60d152fl64f0_1440x690.png)

## 多说一句

这个问题的场景其实是不友好的。良好的编程，不应该再2个分离的文件里设置全局变量，让2个文件通信。

所以会有浏览器端的AMD譬如requirejs去保证顺序和依赖。

也有CMD或es6 module直接引入依赖文件的变量，再通过webpack等打包工具将他们压缩成一个脚本。

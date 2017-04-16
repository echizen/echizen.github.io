---
layout: post
title: "web component漫谈"
description: "web component, Shadow DOM, Custom Elements, HTML Imports, HTML Templates"
category: tech
tags: [html, js]
---
{% include JB/setup %}

## 背景

- 组件化
- 隔离：样式、js脚本

## 内容

2011就有相关文档了

- Shadow DOM
- Custom Elements
- HTML Imports
- HTML Templates

## 原始的组件

[http://codepen.io/robdodson/pen/rCGvJ](http://codepen.io/robdodson/pen/rCGvJ)

[https://github.com/echizen/web-component-demo/blob/master/old/slider.html](https://github.com/echizen/web-component-demo/blob/master/old/slider.html)

需要写html、css、js代码。

引用组件：

- 最不便捷的方式：copy paste
- 使用js模板语言，通过js动态插入html，css单独引入或者通过js动态插入

## web components

### template

作用入名，模板标签，不会被浏览器展现成用户可见的内容，内部内容可以被后续使用。内部可以放置css、html、js

譬如这个slider的template：

    <template>
      <style>
        * {
          -webkit-box-sizing: border-box;
          -moz-box-sizing: border-box;
          -ms-box-sizing: border-box;
          box-sizing: border-box;
        }
        
        #slider {
          max-width: 600px;
          text-align: center;
          margin: 0 auto;
        }
        
        #overflow {
          width: 100%;
          overflow: hidden;
        }
        
        #slides .inner {
          width: 400%;
        }
        
        #slides .inner {
          -webkit-transform: translateZ(0);
          -moz-transform: translateZ(0);
          -o-transform: translateZ(0);
          -ms-transform: translateZ(0);
          transform: translateZ(0);
        
          -webkit-transition: all 800ms cubic-bezier(0.770, 0.000, 0.175, 1.000);
          -moz-transition: all 800ms cubic-bezier(0.770, 0.000, 0.175, 1.000);
          -o-transition: all 800ms cubic-bezier(0.770, 0.000, 0.175, 1.000);
          -ms-transition: all 800ms cubic-bezier(0.770, 0.000, 0.175, 1.000);
          transition: all 800ms cubic-bezier(0.770, 0.000, 0.175, 1.000);
        
          -webkit-transition-timing-function: cubic-bezier(0.770, 0.000, 0.175, 1.000);
          -moz-transition-timing-function: cubic-bezier(0.770, 0.000, 0.175, 1.000);
          -o-transition-timing-function: cubic-bezier(0.770, 0.000, 0.175, 1.000);
          -ms-transition-timing-function: cubic-bezier(0.770, 0.000, 0.175, 1.000);
          transition-timing-function: cubic-bezier(0.770, 0.000, 0.175, 1.000);
        }
        
        #slides img {
          width: 25%;
          float: left;
        }
        
        #slide1:checked ~ #slides .inner {
          margin-left: 0;
        }
        
        #slide2:checked ~ #slides .inner {
          margin-left: -100%;
        }
        
        #slide3:checked ~ #slides .inner {
          margin-left: -200%;
        }
        
        #slide4:checked ~ #slides .inner {
          margin-left: -300%;
        }
        
        input[type="radio"] {
          display: none;
        }
        
        label {
          background: #CCC;
          display: inline-block;
          cursor: pointer;
          width: 10px;
          height: 10px;
          border-radius: 5px;
        }
        
        #slide1:checked ~ label[for="slide1"],
        #slide2:checked ~ label[for="slide2"],
        #slide3:checked ~ label[for="slide3"],
        #slide4:checked ~ label[for="slide4"] {
          background: #333;
        }
        #slides ::content img {
          width: 25%;
          float: left;
        }
      </style>
      <div id="slider">
        <input checked="" type="radio" name="slider" id="slide1" selected="false">
        <input type="radio" name="slider" id="slide2" selected="false">
        <input type="radio" name="slider" id="slide3" selected="false">
        <input type="radio" name="slider" id="slide4" selected="false">
        <div id="slides">
          <div id="overflow">
            <div class="inner">
              <content select="img"></content>
            </div> <!-- .inner -->
          </div> <!-- #overflow -->
        </div>
        <label for="slide1"></label>
        <label for="slide2"></label>
        <label for="slide3"></label>
        <label for="slide4"></label>
      </div>
    </template>

### Shadow DOM

如何将template的内容渲染到页面上呢，就需要借助shadow dom。

shadow dom的内部的内容默认不会展现在chrome的element里，需要在settings中勾选'Show user agent shadow DOM'。

想想我们的`<video>`标签，在element里只能看到这个标签，但是实际上展示的界面有播放键、进度条，这些元素都被隐藏了，其实都是shadow dom。
[http://www.bilibili.com/video/av9606649/](http://www.bilibili.com/video/av9606649/)

    // Add the template to the Shadow DOM
    var tmpl = document.querySelector('template');
    var host = document.querySelector('.img-slider');
    var root = host.createShadowRoot();
    root.appendChild(document.importNode(tmpl.content, true));

- Shadow Host： createShadowRoot方法作用的元素，是能被用户在元素审查器里能看到的元素，也是shadow dom插入的父元素
- Shadow Root：createShadowRoot方法返回的dom片段和它的子元素，对用户隐藏，但是是浏览器实际渲染的元素。
- Shadow Boundary：能够实现私有作用域，得益于这个标准，Shadow Root内的html、css都被Shadow Boundary保护为私有状态，与外届环境隔离。（其实还是有接口可以通信的）
- Insertion Points：通过`<content>`，我们可以把外界的内容插入到shadow DOM中，譬如slider组件内的img内容应该由用户自定义，就像给组件传参一样。`<content>`标签使用css选择器来选择来自shadow host和外部的元素插入到shadow DOM中。（但是我发现去除`select="img"`这个选择器在chrome 57下依旧能工作）。其实仅通过`<content>`这个标签扩展性不足，没法处理多种类型多个位置的子元素的情况，所以还有更强大的`slot`规范：

譬如模板：

    <template id="contact-template">
      <b>Name</b>: <slot name="fullName"></slot><br>
      <b>Email</b>: <slot name="email">Unknown</slot><br>
      <b>Address</b>: <slot name="address">Unknown</slot>
    </template>
    
调用：

    <ul id="contacts">
      <li>
        <span slot="fullName">Commit Queue</span>
        (<a slot="email" href="mailto:commit-queue@webkit.org">commit-queue@webkit.org</a>)<br>
        <span slot="address">One Infinite Loop, Cupertino, CA 95014</span>
      </li>
    </ul>

最终渲染：

    <ul id="contacts">
      <li>
        <!--shadow-root-start-->
        <b>Name</b>:
        <slot name="fullName">
          <!--slot-content-start-->
            <span slot="fullName">Commit Queue</span>
          <!--slot-content-end-->
        </slot><br>
        <b>Email</b>:
        <slot name="email">
          <!--slot-content-start-->
            <a slot="email" href="mailto:commit-queue@webkit.org">commit-queue@webkit.org</a>
          <!--slot-content-end-->
        </slot><br>
        <b>Address</b>:
        <slot name="address">
          <!--slot-content-start-->
            <span slot="address">One Infinite Loop, Cupertino, CA 95014</span>
          <!--slot-content-end-->
        </slot>
        <!--shadow-root-end-->
      </li>
    </ul>


### Custom Elements

接下来就是要定义自定义标签元素，将我们的组件封装成一个含义丰富的标签，供使用方调用：

    var tmpl = document.querySelector('template');

    // Create a prototype for a new element that extends HTMLElement
    var ImgSliderProto = Object.create(HTMLElement.prototype);

    // Setup our Shadow DOM and clone the template
    ImgSliderProto.createdCallback = function() {
      var root = this.createShadowRoot();
      root.appendChild(document.importNode(tmpl.content, true));
    };

    // Register our new element
    var ImgSlider = document.registerElement('img-slider', {
      prototype: ImgSliderProto
    });
    
Custom Elements有2点要求：

- 名字含有连接符`-`，用来和原生标签区分
- 必须继承HTMLElement的prototype

### html import
 
    <link rel="import" href="myfile.html">
    
chrome 49开始已支持。

    <!DOCTYPE html>
    <html>
    <head>
      <title>slider</title>
      <link rel="import" href="./slider.html">
    </head>
    <body>
      <script type="text/javascript">

        var sliderDom = document.querySelector('link[rel="import"]').import
        // // Grab our template full of slider markup and styles
        var tmpl = sliderDom.querySelector('template');

        // Create a prototype for a new element that extends HTMLElement
        var ImgSliderProto = Object.create(HTMLElement.prototype);

        // Setup our Shadow DOM and clone the template
        ImgSliderProto.createdCallback = function() {
          var root = this.createShadowRoot();
          root.appendChild(document.importNode(tmpl.content, true));
        };

        // Register our new element
        var ImgSlider = document.registerElement('img-slider', {
          prototype: ImgSliderProto
        });
      </script>

        <img-slider>
          <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/rock.jpg" alt="an interesting rock">
          <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/grooves.jpg" alt="some neat grooves">
          <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/arch.jpg" alt="a rock arch">
          <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/sunset.jpg" alt="a dramatic sunset">
        </img-slider>
    </body>
    </html>
 

## 示例

完整demo：

[https://github.com/echizen/web-component-demo/blob/master/new-web-components/index.html](https://github.com/echizen/web-component-demo/blob/master/new-web-components/index.html)

[https://github.com/echizen/web-component-demo/blob/master/new-web-components/slider-component.html](https://github.com/echizen/web-component-demo/blob/master/new-web-components/slider-component.html)
 
## 调用

相当的简洁

    <img-slider>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/rock.jpg" alt="an interesting rock">
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/grooves.jpg" alt="some neat grooves">
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/arch.jpg" alt="a rock arch">
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/5689/sunset.jpg" alt="a dramatic sunset">
    </img-slider>

## 生命周期

在这个 API 的基础上，Web Components 标准提供了一系列控制自定义元素的方法。我们来一一看下：

一个自定义元素会经历以下这些生命周期：

- 注册前创建
- 注册自定义元素定义
- 在注册后创建元素实例
- 元素插入到 document 中
- 元素从 document 中移除
- 元素的属性变化时

这个是很重要的内容，开发者可以在注册新的自定义元素时指定对应的生命周期回调来为自定义元素添加各种自定义的行为，这些生命周期回调包括了：

- createdCallback 自定义元素注册后，在实例化之后会调用，通常多用于做元素的初始化，如插入子元素，绑定事件等。
- attachedCallback 元素插入到 document 时触发。
- detachedCallback 元素从 document 中移除时触发，可能会用于做类似 destroy 之类的事情。
- attributeChangedCallback 元素属性变化时触发，可以用于从外到内的通信。外部通过修改元素的属性来让内部获取相关的数据并且执行对应的操作。

## 兼容性

兼容性不是很乐观，5年过去了，但是只有chrome和opera实现的最好，其他浏览器还是没有开始支持。

Firefox默认禁用Web Components。启用的步骤是，前往 about:config 页面，注意取消出现的所有警告，搜索名为 dom.webcomponents.enabled 的配置，设置为 true。

![http://s2.mogucdn.com/mlcdn/c45406/170410_72a8866g7kc8f6dg3ke97eal3jfa0_2236x1076.png](http://s2.mogucdn.com/mlcdn/c45406/170410_72a8866g7kc8f6dg3ke97eal3jfa0_2236x1076.png)

![http://s2.mogucdn.com/mlcdn/c45406/170410_13ei0agaj8acf1g49bl81ae41ibad_2232x936.png](http://s2.mogucdn.com/mlcdn/c45406/170410_13ei0agaj8acf1g49bl81ae41ibad_2232x936.png)

但是Mozilla 和 Google 开发了强大的polyfill库，来实现主流浏览器的支持：[https://www.polymer-project.org/](https://www.polymer-project.org/)


## 问题

为啥发展的这么慢？虽然已被w3c指定为标准，但是只有chrome和opera支持的最好，5年过去了，其他浏览器还是没有开始真正支持，是否被抛弃

# 参考

[https://css-tricks.com/modular-future-web-components/](https://css-tricks.com/modular-future-web-components/)

[https://github.com/w3c/webcomponents](https://github.com/w3c/webcomponents)

[https://www.html5rocks.com/zh/tutorials/webcomponents/imports/](https://www.html5rocks.com/zh/tutorials/webcomponents/imports/)

以下我还未认真看：

[https://developer.mozilla.org/zh-CN/docs/Web/Web_Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components)

[https://segmentfault.com/a/1190000006745770](https://segmentfault.com/a/1190000006745770)
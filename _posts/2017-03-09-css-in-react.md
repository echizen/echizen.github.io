---
layout: post
title: "react 项目中的css处理"
description: "react 项目中的css处理, css加载，style-loader作用，extract-text-webpack-plugin抽取css，css 模块化， css in js, css module, npm包组件里的css处理"
category: tech
tags: [react, css, 工程化]
---
{% include JB/setup %}

## css加载

### style-loader

一般性配置：

js file:

    import './message.css';
    
webpack:

     module: {
        loaders: [
            {
                test: /\.(scss|css)$/,
                loaders: ['style', 'css', 'postcss']
            }
    	 ]
    },

    postcss: function () {
        return [require('autoprefixer'), require('precss'), require('postcss-calc')];
    }
    
先引入postcss-loader处理，譬如precss将sass转成css，autoprefixer添加兼容性前缀。再由css-loader加载require的样式文件，最后style-loader将这些css转变成会插入到页面`<style>`里的样式代码。

最终`<head>`标签下会有非常多的`<style>`标签，来安放这些css文件里的样式代码。

style-loader是每次遇到import或者require css类文件的语句时，就向head里添加一个`style`标签，来安放css代码。这样频繁操作dom很低效，且新的样式规则的插入会导致浏览器频繁的重绘。并且一开始没有css样式，这样一点点插入，首屏渲染时间也会加长

对于我这种之前在项目中都是单独在html文件里通过`<link>`引入css文件的人来说，这种方式简直逆天。

不过后来我用Timeline查看了下各项性能数据指标，之能说理论如前所述，但是对于pc页面在强大的chrome下的渲染来看，这种性能损耗差别比较小。

### extract-text-webpack-plugin

anyway，我还是要说一说可以自动打包css成单独文件的extract-text-webpack-plugin，这个不止是用在react项目中，用在vue\angular都可以：

[https://github.com/webpack-contrib/extract-text-webpack-plugin/blob/webpack-1/README.md](https://github.com/webpack-contrib/extract-text-webpack-plugin/blob/webpack-1/README.md)

如果希望样式通过`<link>`引入，而不是放在`<style>`标签内。使用extract-text-webpack-plugin插件可以帮我们独立出css文件。

    var ExtractTextPlugin = require("extract-text-webpack-plugin"); 
    module.exports = {    
    // ...   
      module: {         
        loaders: [                        
            {
              test: /\.css$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader")
            },
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader")
            }
      },     
      plugins: [         
        new ExtractTextPlugin("[name].css")     
      ] 
    }
    
ExtractTextPlugin.extract可以有三个参数。
- 第一个参数是可选参数，传入一个loader，当css样式没有被抽取的时候可以使用该loader。
- 第二个参数则是用于编译解析的css文件loader，很明显这个是必须传入的，就像上述例子的css-loader。
- 第三个参数是一些额外的备选项。

插件也支持所有独立样式打包成一个css文件。增加多一个参数，指定allChunks为true。

new ExtractTextPlugin("style.css", {allChunks: true})

引入时，只需要在html里引入合并后的css文件，注意并不需要在js里再require或者import这个文件了。

#### 优点

- css是单独文件，放到head里优先加载生效
- js文件会变小
- 减少了js操作dom插入style标签带来的损耗
- 不频繁修改的css能够利用缓存获取

## css 模块化

那么在模块化的项目开发里如何处理css呢？

项目中遇到很多css相关的头疼的问题：

1. 样式覆盖，多人协作更严重，css样式默认是全局作用域的。
2. 为了尽量减少模块样式相互覆盖，我们可能采用更深层级的样式选择器，这样css渲染效率变低了，而且权值这个没法保证，可能有些class权值一致，因为引入先后顺序，后面的定义生效了，但是下次打包css顺序变了，还是会出现问题，风险存在。

看看业内处理这个问题主要有2种方案：1.css in js. 2.css module

### css in js

彻底抛弃 CSS，使用 JS 或 JSON 来写样式。Radium，jsxstyle，react-style 属于这一类。

#### react-style

譬如[react-style](https://github.com/js-next/react-style):

定义样式：

    var StyleSheet = require('react-style') 
    var styles = StyleSheet.create({
        foo: {
            color: 'red',       
            backgroundColor: 'white'     
          } 
    })

引入样式：

    var React = require('react') 
    class HelloWorld extends React.Component{   
      render() {     
        var dynamicStyles = {color: this.props.color}     
        return <div styles={[styles.foo, dynamicStyles]}>Hello, world!</div>   
      } 
    }

最终样式会转成dom的`inline style`。

#### 组件js代码里自行处理

还有一种不能算css 模块化的内容，但是是`css in js`的一种做法:

譬如material-ui，直接在组件内定义内容为css样式的变量，然后将其运用在组件的style标签上。

[https://github.com/callemall/material-ui/blob/master/src/Checkbox/Checkbox.js](https://github.com/callemall/material-ui/blob/master/src/Checkbox/Checkbox.js)

#### radium

radium相对于material-ui那种自行处理，做了一些封装，来提高开发效率

[https://github.com/FormidableLabs/radium](https://github.com/FormidableLabs/radium)

    var Radium = require('radium');
    var React = require('react');
    var color = require('color');

    @Radium
    class Button extends React.Component {
      static propTypes = {
        kind: React.PropTypes.oneOf(['primary', 'warning']).isRequired
      };

      render() {
        return (
          <button
            style={[
              styles.base,
              styles[this.props.kind]
            ]}>
            {this.props.children}
          </button>
        );
      }
    }

    var styles = {
      base: {
        color: '#fff',
        ':hover': {
          background: color('#0074d9').lighten(0.2).hexString()
        }
      },

      primary: {
        background: '#0074D9'
      },

      warning: {
        background: '#FF4136'
      }
    };

- 优点：是能给 CSS 提供 JS 同样强大的模块化能力；
- 缺点是不能利用成熟的 CSS 预处理器（或后处理器） Sass/Less/PostCSS，:hover 和 :active 伪类处理起来复杂

### css module

[https://github.com/css-modules/css-modules](https://github.com/css-modules/css-modules)

CSS Modules 内部通过 ICSS 来解决样式导入和导出这两个问题。分别对应 :import 和 :export 两个新增的伪类。

    :import("path/to/dep.css") {
        localAlias: keyFromDep;
        /* ... */
    }
    :export {
        exportedKey: exportedValue;
        /* ... */
    }
    
但直接使用这两个关键字编程太麻烦，实际项目中很少会直接使用它们，我们需要的是用 JS 来管理 CSS 的能力。结合 Webpack 的 css-loader 后，就可以在 CSS 中定义样式，在 JS 中导入

webpack配置：

    // webpack.config.js
    css?modules&localIdentName=[name]__[local]-[hash:base64:5]
    
给css-loader加上 modules参数 即为启用，localIdentName 是设置生成样式的命名规则。

定义css：

    /* components/Button.css */ 
    .normal { /* normal 相关的所有样式 */ } 
    .disabled { /* disabled 相关的所有样式 */ }
    
使用：

    /* components/Button.js */ 
    import styles from './Button.css'; 
    console.log(styles); 
    buttonElem.outerHTML = `<button class=${styles.normal}>Submit</button>`
    
生成的 HTML 是：

    <button class="button--normal-abc53">Submit</button>
    
##### 命名空间

css module里默认样式都是局部的，相当于给每个 class 名外加加了一个 :local，以此来实现样式的局部化。如果想切换到全局模式，需要指定 :global。

    .normal {
      color: green;
    }

    /* 以上与下面等价 */
    :local(.normal) {
      color: green; 
    }

    /* 定义全局样式 */
    :global {
      .link {
        color: green;
      }
      .box {
        color: yellow;
      }
    }

##### 样式复用

css module使用composes来复用样式：

    .className {
      color: green;
      background: red;
    }

    .otherClassName {
      composes: className;
      color: yellow;
    }
    
可以复用多个class的样式： `composes: classNameA classNameB`。

可以compose一个来源于其他文件的class:

    .otherClassName {
        composes: className from "./style.css";
    }

composes 其实不是复用了具体的样式属性和值，而是会编译成多个class

    import styles from './styles.css';

    buttonElem.outerHTML = `<button class=${styles.otherClassName}>Submit</button>`

这段代码会转化成：

    <button class="button--className-daf62 button--otherClassName-abc53">Submit</button>
    
    
##### CSS，JS变量共享

使用`:export` 关键字可以把 CSS 中的 变量输出到 JS 中。

譬如在js中读取sass的变量：

    /* config.scss */
    $primary-color: #f40;

    :export {
      primaryColor: $primary-color;
    }
    /* app.js */
    import style from 'config.scss';

    // 会输出 #F40
    console.log(style.primaryColor);

##### 优势

- 样式默认都是局部的，解决了命名冲突和全局污染问题
- 只需引用组件的 JS 就能搞定组件所有的 JS 和 CSS
- css代码依旧是css，学习成本低，符合长期形成的开发习惯
- 可以与预处理器共同使用

### shadow dom

https://developer.mozilla.org/en-US/docs/Web/Web_Components/Shadow_DOM


## npm包组件里的css输出

把组件做成npm提供给其他人使用，参考现有的一些组件。主要有2种方式：

1. 使用webpack将css合并成一个文件，js也合并成一个文件输出
2. 仍按原组件结构，不拆分文件，对外提供入口文件供引入。这时候js和jsx文件会用babel编译一下。

我更喜欢2，这样源码能够被查看调试。但是途中遇到一些需要解决的问题：

1. sass、less文件的处理：用相应的编译器转成css。可以用webpack配置多入口（每个入口就一个css文件）的方式转成css，这其实不是在把webpack当合并打包工具使用了。。。我嫌麻烦，用了gulp。
2. 用了gulp转成css后，原文件中的引用语句： `import 'xxx.scss'`会找不到。只能用gulp-replace在将文件里的'.scss'替换成'.css'。
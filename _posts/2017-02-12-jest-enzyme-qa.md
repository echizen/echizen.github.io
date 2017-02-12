---
layout: post
title: "使用jest+enzyme进行react项目测试 - 疑难杂症篇"
description: "解决在webpack的resolve.alias中配置的内容jest找不到：ReferenceError: React is not defined。jest测试代码的调试。enzyme simulate input change。jest mock fetch 和 jsonp。jest 存储html格式的Snapshot。enzyme获取非root的state。enzyme simulate antd checkbox change。enzyme simulate 清除antd 的timePicker选择器值。jest 对于动态插入的script标签引入的文件处理
category: tech
tags: [测试, react]
---
{% include JB/setup %}

恩，最要命的是配置环境的时候。虽然几乎所有库都会在官网首页介绍他们的配置多么多么简单，just one step，就可以跑起来。恩，确实按照套路，demo是能跑起来，但是要能不再配置其他内容，达到能随心所欲跑自己的任何一个测试文件，还是路漫漫啊。

## 1.解决在webpack的resolve.alias中配置的内容jest找不到

### 报错

`ReferenceError: React is not defined`

### 原因

原文件和测试文件里都引入了react:

    import { Component } from 'react’; ,

 组件是这样定义的：
 
    class FileUploadInput extends Component { render(){}
 
 并没有显式的使用React变量，但是经过babel编译后会成为`React.createElement`，使用了React对象。
 
而我的webpack配置里，为了提高打包和编译速度、减小压缩包体积，将react作为外部变量引入的：

    externals: {
        'react': 'React',
        'react-dom': 'ReactDOM'
    }

所以我也要在jest的配置里将React作为外部的一个全局变量引入。

### 解决方案

在package.json里jest的配置里配置"setupFiles"，指向setup.js。setup.js中声明React。

    import React from 'react';
    if (typeof window !== 'undefined') {
      window.React = React;
    }


## 2. 调试

恩，测试代码也是需要调试的，需要知道测试代码本身哪里有问题。

### node --inspect

我先是将node升级到7，使用`node --inspect`: `node --inspect --debug-brk ./node_modules/jest-cli --runInBand`，速度很快，浏览器秒开并断点定位到首行。但是后续进入不了自己打的debugger断点，因为还没有进入到编译打包的步骤，测试文件和原文件都还没在浏览器source里出现，没法手动加断点。也就是没法打自己需要的断点。后来查了下这是`node --inspect`的已知问题：[https://github.com/facebook/jest/issues/1652](https://github.com/facebook/jest/issues/1652)。

### node-inspector

安装node-inspector，运行
`node-debug --nodejs --harmony ./node_modules/jest-cli/bin/jest.js —runInBand`，然后运行`node-inspector`，再按终端提示操作。

可以成功设置断点调试，但是浏览器定位、打包速度超级慢，都是分钟级别的。最终受不了这速度，放弃。

### 不完美的终篇，console.log

最后还是使用原始的`console.log`调试的。这个方式并不满意，但是确是这3种方式中操作起来最快的。

## 3.改变input值：

    wrapper.find('input').simulate('change', {target: {value: 'My new value'}});

### 4.mock fetch 和 jsonp

坑爹的nock 只能mock http(s).request, 不能mock fetch。被[http://cn.redux.js.org/docs/recipes/WritingTests.html](http://cn.redux.js.org/docs/recipes/WritingTests.html)带到沟里去了。

我引用的第三方mock请求的组件是`fetch-mock`;

先封装个统一的mock方法：

    const fetchMock = require('fetch-mock');
    import { HOST } from '../src/util/api.js';

    export function mockRequest(path,res){
      let reg = new RegExp(`${HOST}${path}.*`);
      return fetchMock.get( reg, {
        body: {
          status:{
            code: "0",
            detail: "成功",
            msg: "success"
          },
          result: res
        },
        status: 200
      })
    }
    
调用：

    import { mockRequest } from '../mockRequest.js';
    import { multiData } from './mockModuleData.js';

    describe('DataEdit render', () => {
      afterEach(fetchMock.restore)
      it('renders DataEdit correctly', () => {
        mockRequest('/pageModule/getData', multiData);
        return wrapper.node.fetchData(moduleInfo.moduleId).then(()=>{
          expect(toJson(wrapper)).toMatchSnapshot();
        })
      });
    })

jsonp我原文件使用的是`fetch-jsonp`库提供的`fetchJsonp`方法，由于jsdom不是真实的浏览器不能真的插入script请求js，`fetch-mock`也mock不了jsonp。

所以我先将fetchJsonp mock 成fetch，再由`fetch-mock`统一Mock。


## 5. 存储html格式的Snapshot

有些组件如antd 和 rc的modal，调用click事件后才会出来真正的ui结构，但是这个dom并不是打印在Modal组件这层作为子元素，而是作为body的直接子元素，就是这段ui结构直接插在body下。

这时候我们只能通过选择器选到modal。但是jest的`toMatchSnapshot`本身只是检查这次的快照和上次有没有区别，并不关心快照内容是什么和快照是怎么生成的，所以我们完全可以将选择器选中内容的innerHTML作为快照。需要做的就是格式化这段html，这样有改动时方便精确定位到变更的位置。

我使用了一个开源的`beautifyHtml`组件来格式化html。

    import style_html from '../beautifyHtml.js';
    let ele = document.querySelector('.module-data-modal').innerHTML;
    expect(style_html(ele)).toMatchSnapshot();

## 6. enzyme获取非root的state

enzyme官方文档说明state()方法只能在mount的根组件上调用，但是我们经常通过根组件选择子组件，需要获取子组件的state，这是后可以通过node节点来获取，譬如：

    const wrapper = mount(InputEditView);
    const InputEditCom = wrapper.find('InputEdit').first();
    expect(InputEditCom.node.state.validateStatus).toBe('');

其实神奇的node获得的是ReactComponent，可以获取组件上所有内容，可以打印出来看一下，能做很多事。

## 7.触发antd checkbox值的改变

    wrapper.find('input[type="checkbox"]').first().simulate('change', { target: { checked: true } });

并不是想象中的simulate(‘click’)

## 8.清除antd 的timePicker选择器值

清除时间选择器是mouseDown事件而不是click事件，还得去看rc-time-picker源码：[https://github.com/react-component/time-picker/blob/master/tests/Header.spec.jsx](https://github.com/react-component/time-picker/blob/master/tests/Header.spec.jsx)

## 9.动态插入的script标签引入的文件处理

我的项目里还有一种需要mock的场景，就是我为了保证线上页面js体积最小化，有些在特定场景下需要的用到的第三方库，我是使用动态插入script标签到页面中的方式来加载这些文件的。这些第三方库多是抛出了个全局变量供我的代码使用。但是在测试环境中，jsdom并不是浏览器，动态插入script加载个js文件并不能成功。

这时候我需要先引入这个会被动态插入的js，先抛出来作为全局变量。

    import MEditor from './mockEditor.js';
    describe('Editor', () => {
      const id = 'testEditor123';
      function creat(){
        return (<Editor value = { '初始化内容' }
                  handleChange={ jest.fn() }
                  id = { id }
                />)
      }

      // insert editor js to window
      beforeEach(() => {
        window.MEditor = MEditor;
      })

      it('call handleChange fn', () => {
        const view = creat()
        const wrapper = mount(view);

        const newValue = "呵呵那个呵呵";
        const inputArea = document.querySelector(`#${id}`);
        expect(inputArea.length).toBe(1);
        inputArea.innerHTML = newValue;
        expect(wrapper.props().handleChange).toBeCalledWith( newValue );
      });

    })
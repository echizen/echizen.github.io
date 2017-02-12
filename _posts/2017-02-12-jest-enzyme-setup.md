---
layout: post
title: "使用jest+enzyme进行react项目测试 - 搭建篇"
description: "搭建jest+enzyme环境进行react项目测试。package.json中moduleFileExtensions、testRegex、moduleNameMapper、setupFiles配置。jest setupFiles使用场景及作用。jest对react组件测试demo"
category: tech
tags: [测试, react]
---
{% include JB/setup %}

以自己的项目为例，介绍一下这套结构的搭建，项目使用了jest、enzyme、enzyme-to-json。

### 安装所需依赖

    npm install jest --save-dev
    npm install enzyme --save-dev
    npm install enzyme-to-json --save-dev

### 配置

package.json可以配置些基础配置，我的配置有：

    "jest": {
      "moduleFileExtensions": [
        "js",
        "jsx",
        "json"
      ],
      "testRegex": ".*\\.spec\\.js$",
      "collectCoverageFrom": [
        "src/**/*.{js,jsx}",
        "src/**/**/*.{js,jsx}",
        "!**/node_modules/**"
      ],
      "moduleNameMapper": {
        "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/spec/__mocks__/fileMock.js",
        "\\.(css|scss)$": "<rootDir>/spec/__mocks__/styleMock.js"
      },
      "setupFiles": [
        "<rootDir>/spec/setup.js"
      ]
    }

- moduleFileExtensions： 被测试的文件里依赖的文件后缀，我的项目使用webpack打包，也配置了文件后缀，这样依赖的时候只用写`require 文件名`，工具会根据的你配置去找相应后缀的文件。
- testRegex：正则表示的测试文件，测试文件的目录有很多种规划：1.单独放在测试文件夹。2.放在原文件夹，和原文件相同的位置。我是组件都放在测试文件夹，但是业务代码的测试还放在原文件附近，所以我通过文件名去匹配，我所有测试文件格式都是xxx.spec.js。
- collectCoverageFrom：生成测试覆盖报告时检测的覆盖文件，我的都位于src下的js和jsx文件，但是要去除node_modules下所有文件，因为其会被作为依赖在原文件里处处引入。
- moduleNameMapper： mock了图片、css、字体、音视频文件。
- setupFiles：配置文件，在运行测试案例代码之前，jest会先运行这里的配置文件来初始化你指定的测试环境。


setupFiles是个非常有用的项目，可以解决很多问题，在这里你可以为全局的window或global对象上绑东西，配置自己代码要运行的环境。譬如我在setup.js里配置了全局的react，mock了localstorage，随着测试的进行，以后应该还会有其他需求加在这里。

    import React from 'react';
    if (typeof window !== 'undefined') {
      window.React = React;
      window.localStorage = ( function storageMock() {
        var storage = {};
        return {
          setItem: function(key, value) {
            storage[key] = value || '';
          },
          getItem: function(key) {
            return key in storage ? storage[key] : null;
          },
          removeItem: function(key) {
            delete storage[key];
          },
          get length() {
            return Object.keys(storage).length;
          },
          key: function(i) {
            var keys = Object.keys(storage);
            return keys[i] || null;
          }
        }
      })()
    }


至此，我的测试环境就搭好了。剩下的就是单元测试代码的编写了。

### 组件测试代码结构

你要测试的组件或者函数必须在原文件里export出来，测试文件才能引用到，置于原文件的依赖，只要不是被mock的文件，都会真实的加载执行。所以一个react组件测试的结构类似这样：

    import React from 'react';
    import { render, mount, shallow } from 'enzyme';
    import toJson from 'enzyme-to-json';
    import { component } from 'component_path';


    describe('component test', () => {

      it('test one aspect', () => {
        
      });
    })
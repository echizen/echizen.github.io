---
layout: post
title: "使用jest+enzyme进行react项目测试 - debug篇"
description: "jest debug, vscode jest debug, jest debug single file"
category: tech
tags: [jest, react]
---
{% include JB/setup %}


这套环境是node服务在跑，测试代码也会出bug，也需要完善的调试环境。

在经历了借助工具`node-inspector`，但是太慢；node 7 自带的`node --inspect`只能定位到`--debug-brk`指定的断点处，自己设置的debugger断点无效；以及很长一段时间的`console.log`大法。

后来终于从另一个角度解决了问题，借助编辑器IDE的强大功能。使用vs code强大的调试功能，简直不能更好用。简直跟chrome调试手感没啥差别，连断点编辑功能都有~

如果使用的不是vs code，请先下载编辑器： [https://code.visualstudio.com/](https://code.visualstudio.com/)。不要害怕换了不习惯，体验还是很好的。找个风和日丽的日子，有足够时间适应一下。官网有debuggger文档，但是可以不看，因为跟chrome调试操作差不多。

### 配置

点击debugger的小虫子icon，进入debug模式后，项目根目录下的`.vscode`下会出现`launch.json`文件，这是debug的配置文件。

jest的配置内容如下：

    {
      "version": "0.2.0"
      , "configurations": [{
          "type": "node"
          , "request": "launch"
          , "name": "jest"
          , "program": "${workspaceRoot}/node_modules/jest-cli/bin/jest.js"
          , "stopOnEntry": false
          , "args": ["--runInBand"]
          , "cwd": "${workspaceRoot}"
          , "preLaunchTask": null
          , "runtimeExecutable": null
          , "runtimeArgs": [
            "--nolazy"
          ]
          , "env": {
            "NODE_ENV": "development"
          }
          , "externalConsole": false
          , "sourceMaps": false
          , "outDir": null
        }
      ]
    }


其中有几点需要注意：
 - "type": "node"。指定是node类型文件调试
 - "program": "${workspaceRoot}/node_modules/jest-cli/bin/jest.js"。指定了入口文件，在这里就是jest的入口文件，平时通过终端运行jest也调用的是这个脚本。
 - "args": ["--runInBand"]。命令携带的参数，jest的特点是多进程并发运行不同测试案例，达到快速的效果。但是这样对调试来说是没法进行的。这个参数保证了使用一个进程运行所有代码。
 - "env": 指定了运行时的一些环境变量
 - "sourceMaps": 调试时，部分运行文件是压缩状态的，使用这个可以方便查看。但是我没开启也是可以的，大部分要设断点的都是未压缩状态的代码

这些配置相当于运行脚本：`node --debug-brk=13671 --nolazy node_modules/jest-cli/bin/jest.js --runInBand`。 `--debug-brk=num`就是你设置的断点处，编辑器给你转成了脚本参数

### 调试单个文件

上面的配置其实是运行了`jest`的命令，会一股脑把所有测试文件都跑一遍。实际场景中我们大多数是要调试某个测试文件。

我们发现args是命令携带的参数，如果要运行某个测试文件，命令形如`jest oneFileName.spec.js`。

所以配置一下args加个具体的文件参数就好啦~

`"args": ["--runInBand", "oneFileName.spec.js"]` 

之前一直带上了`--`符，哭晕。。。


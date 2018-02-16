---
layout: post
title: "node调用jest"
description: "node调用jest"
category: tech
tags: [jest]
---
{% include JB/setup %}

因为jest官网没有介绍node里调用jest的文档，只给出了cli和npm命令通过配置文件调用的方式，故将通过阅读源码获取的node调用jest的方式记录下来。

# 入口

jest执行测试需要调用内部接口`runCLI`

    import { runCLI } from 'jest'
    runCLI(argv, projects)

- argv(object): 配置参数
- projects(array): 被测试文件所在文件夹路径，一般与argv.projects项一致
- return(promise): 返回promise表示测试进行状态

# 配置

上文中的argv参数列表和意义请参考: [Configuring Jest](https://facebook.github.io/jest/docs/en/configuration.html)

argv各项类型: [https://github.com/facebook/jest/blob/master/types/Argv.js](https://github.com/facebook/jest/blob/master/types/Argv.js)

argv参数示例请参考: [https://github.com/facebook/jest/blob/master/packages/jest-config/src/valid_config.js](https://github.com/facebook/jest/blob/master/packages/jest-config/src/valid_config.js)

默认参数：[https://github.com/facebook/jest/blob/master/packages/jest-config/src/defaults.js](https://github.com/facebook/jest/blob/master/packages/jest-config/src/defaults.js)


需要注意的是**如果argv的某一项类型是object，需要做`JSON.stringify`转化成字符串**


主要就是调用时需要找到入口和传参，其他的在Node端运行就没有障碍了。
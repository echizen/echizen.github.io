---
layout: post
title: "npm对包版本的管理"
description: "npm对包版本的管理"
category: tech
tags: [npm]
---
{% include JB/setup %}

使用npm时是否会有这些疑惑：为什么package.json里写的是`"repo": "^x.x.x"`，执行`npm update repo`却不能更新到最新版？各种包之间互相依赖，每个依赖版本限制都不一样，那么最终到底会下载哪个版本？各种包依赖版本有冲突时怎么处理？

# package.json中的版本指定

package.json形如:

	{
	  "name": "repo-name",
	  "version": "1.0.0",
	  "dependencies": {
	    "react": "^15.2.1",
	    "react-dom": "^15.2.1",
	    "react-redux": "^4.0.0",
	  },
	  "devDependencies": {
	    "autoprefixer": "^6.1.2",
	    "babel-core": "~5.8.20",
	    "babel-eslint": "~4.0.5",
	    "babel-loader": "~5.3.2",
	  }
	}
    
`dependencies`是该库必备依赖包，`devDependencies`是开发环境依赖包，多是一些工具包。每个依赖包都对版本有指定，常见的是`^`和`~`。

对于版本`x1.x2.x3`，举例版本为`1.2.3`。

- `^1.2.3`表示大于等于`1.2.3`形如`1.x.x`的版本，注意虽然是大于指定版本，但是x1值是指定值，他能匹配`1.2.5`，也能匹配`1.3.1`，但是不能匹配`2.0.0`。

- `~1.2.3`表示大于等于`1.2.3`形如`1.2.x`的版本，注意这里x1,x2的值都等于指定值，可以匹配`1.2.5`，但是不可以匹配`1.3.1`、`2.0.0`。

所以最通用的写法`^1.2.3`并不是所有表示大于1.2.3的版本都可以匹配。如果要表示所有大于`1.2.3`的版本都匹配的话得用`>=1.2.3`。但是一般不这么做，这样有很大风险。**大版本的发布一般伴随着api的变更**，x1值的改变如果都去更新的话，向下兼容没做好的库，对于使用者来说会带来bug。

# npm3的扁平化方案

npm3与npm2的对依赖包的解决方案有很大不同。我最近才更新至npm3。。。要及时更新工具啊。

npm2 采用严格树形嵌套的形式组织依赖模块的目录，而 npm3 则尽量扁平化，将依赖模块提升至顶层目录：

![image](https://docs.npmjs.com/images/npm3deps4.png)

**npm安装包的顺序由字符顺序决定，最先被装到顶层目录的依赖包版本由最先依赖他的包决定**

于此同时，npm3也会在终端输出包的结构：

![image](https://docs.npmjs.com/images/tree.png)

如果你想看原始依赖关系，可以使用`npm ls`:

![image](https://docs.npmjs.com/images/npmls.png)

# npm3如何抉择各个库对各种版本包的需求

在上图的例子中，如果此时来了个D v1.0，它依赖的是B v2.0，结构会变成怎样呢？这时候B v2.0不会被安装在顶级目录下，而是继续留在B的node_modules下。

![image](https://docs.npmjs.com/images/npm3deps6.png)

如果再来个E v1.0，他依赖的是B v1.0。由于B v1.0已经在顶层目录了，这时候B v1.0不会再被下载，而是共享顶层目录的B v1.0。

![image](https://docs.npmjs.com/images/npm3deps8.png)

**扁平化后，顶层node_modules目录下的包会被共享**

# 对于包升级的处理

如果此时我们将A v1.0升级到v2.0，A v2.0依赖的是B v2.0，会发生什么呢？

- npm先删除A v1.0
- 下载A v2.0
- Bv1.0 不会被删除因为 E v1.0 仍然依赖它。
- 下载Bv2.0至A的node_modules底下。因为Bv2.0已经占有了顶层目录的位置。

但是注意，此时，对于一个`node_modules`为空的刚加入该项目开发的成员来说，第一次`npm install`后他的目录结构是不同的，他的根目录下是Bv2.0，因为按字母顺序下载，A2.0依赖的是B2.0。所以这样会出现项目成员目录不一致的情况，不过并没有什么关系。不影响项目的运行和最终产物。

如果此时我们再更新E v1.0至E v2.0，它依赖的是B v2.0，这回发生什么呢？

- npm先删除Ev1.0
- 下载Ev2.0
- 删除Bv1.0
- 下载Bv2.0至顶层目录

这时就尴尬了，大家依赖的都是Bv2.0，Bv2.0却处处存在。

![image](https://docs.npmjs.com/images/npm3deps12.png)

此时就需要`npm dedupe`了

# npm dedupe

执行后npm会检查所有包，重新按规则整理，按包的字母顺序整理依赖，对于依赖包指定的包，删除其他包中node_modules私有文件夹下的包。最终产物就是：

![image](https://docs.npmjs.com/images/npm3deps13.png)

# 扁平化方案的优势

资源共享，减少体积。对于webpack这样的打包工具能减少最终产物体积。

对于a依赖`"b":"^1.2.3"`,b最新包是1.9.0。此时npm会在a的node_modules下装上1.9.0的b。如果后继开发中我们引入了c，c的依赖是`"b":"^1.x.x"`这种，但此时b的最先版已经更新到1.9.2了，此时npm会在下载个b1.9.2至c的node_modules下，最终webpack打包时，打了2份不同版本的b。。。

同样情况发生在npm3下，引入c后检查到bv1.9.0满足c对b的依赖条件，便不会再下载b，最终只会打包一份b。

# 扁平化方案的吐槽

这个听Node程序员吐槽过包被扁平化后，不方便调试和查找了。而且为了避免同一资源被多次下载，其实可以通过软链接的方式，而不必要改变原本的包结构。

# 黄金外链

[https://docs.npmjs.com/how-npm-works/npm3](https://docs.npmjs.com/how-npm-works/npm3)

[https://docs.npmjs.com/how-npm-works/npm3-dupe](https://docs.npmjs.com/how-npm-works/npm3-dupe)

[https://docs.npmjs.com/cli/dedupe](https://docs.npmjs.com/cli/dedupe)
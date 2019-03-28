---
layout: post
title: "webpack-性能优化"
description: "webpack 性能优化，提高webpack构建速度，优化webpack构建代码"
category: tech 
tags: ['webpack']
---
{% include JB/setup %}

webpack性能优化分为两方面，一方面是development模式下提高开发环境的构建速度，让你的改动快速的更新到浏览器供调试，另一方面是production模式下提高生产环境的bundle代码质量，减少体积，优化代码的运行时性能。

anyway，我认为在条件允许的情况下，首要优化方案都是**升级到最新版webpack**，毕竟其他措施都是辅助，webpack核心代码的优化才是主力，webpack1的速度和4的是不能比的。

然后webpack4提供了两种mode供配置，每种mode都有一套性能较佳的默认配置，省去了我们很多人工配置的注意点。

# 优化development模式构建性能

* **选择合适的source-map，development用`cheap-module-source-map`**。source-map因为其体积大信息多构建慢，真的是最能影响构建速度的一个方面了。在符合需要的情况下选择一个尽量简单的譬如不包含列信息sourcemap，性能提升看得见。
* **拆项目，减少构建范围**。再怎么优化也扛不住几年时间码起来的巨大的项目，不如拆成几个独立的部分单独构建。
* **配置external，不打包react\react-dom\vue这些**，直接引用cdn资源。这些资源变更少体积大，根本无需实时参与构建，拆出去，构建压力小不少
* **使用happypack**， 让loader可以多进程去处理文件。webpack4对uglifyjs插件都有了多进程支持，但是官方对loader还没有支持多进程,目前happypack仍是一个好选择
* 开启uglifyjs并行压缩（其实默认就是开启的）：
    minimizer: [
          new UglifyJsPlugin({ parallel: true })
    ]

* **配置resolve.modules指定模块获取范围**，减少loader的处理查找范围
* **精确的resolve.extensions**，作用同上
* **配置DLLPlugin**:这个的作用和配置external是异曲同工，通过单独的webpack配置文件指定哪些公用基础库生成dll，譬如vue/vuex/vue-router，通过配置`new webpack.DllPlugin(options)`生成dll的js bundle和dll的manifest.json，manifest.json记录的是这些生成dll模块的map映射表，供后面项目模块import和require引用时查找。然后再在项目的webpack中使用`new webpack.DllReferencePlugin({ manifest: require("path/dll.manifest.json")})`来引入dll文件关系供使用。dllPlugin相对external的好处是更加动态化，如果你某一天真的需要升级其中一个npm包了结合`add-asset-html-webpack-plugin`资源重新打包和路径替换都可以自动处理，而external的资源只能手动处理更新了，不过dllPlugin的配置使用姿势更麻烦。 [配置姿势详情https://webpack.docschina.org/plugins/dll-plugin/](https://webpack.docschina.org/plugins/dll-plugin/)
* 开缓存[]有哪些地方可开
* **配置`bable-loader`的`cacheDirectory:true`**： 或设置指定路径值，当有设置时，指定的目录将用来缓存 loader 的执行结果。之后的 webpack 构建，将会尝试读取缓存，来避免在每次执行时，可能产生的、高性能消耗的 Babel 重新编译过程(recompilation process)。（如果设置了一个空值 (loader: 'babel-loader?cacheDirectory') 或者 true (loader: babel-loader?cacheDirectory=true)，loader 将使用默认的缓存目录 node_modules/.cache/babel-loader，如果在任何根目录下都没有找到 node_modules 目录，将会降级回退到操作系统默认的临时文件目录。）

# 优化production模式bundle代码

* **配置`optimization.splitChunks`**：作用是以一定规则划分多个bundle文件，将每个文件体积减小，更有价值的是划分规则，譬如你可以将node_modules下的文件都划分到一个独立的chunks里，npm包里的内容变更频率很低，这样每次构建发布这部分都是不变的内容，网站使用时就能充分利用浏览器缓存加速了。[详情请看https://juejin.im/post/5b99b9cd6fb9a05cff32007a](https://juejin.im/post/5b99b9cd6fb9a05cff32007a) 。提取出第三方库代码和异步加载的代码，webpack会根据下述条件自动进行代码块分割（你也可以根据自己的需求配置更佳的规则）：
    * 新代码块可以被共享引用，或者这些模块都是来自node_modules文件夹里面
    * 新代码块大于30kb（min+gziped之前的体积）
    * 按需加载的代码块，并行请求最大数量应该小于或者等于5
    * 初始加载的代码块，并行请求最大数量应该小于或等于3
* **配置`optimization.runtimeChunk : true  ||  "multiple"`**，将webpack动态化的脚手架代码单独拆出，可以让根据文件内容hash生成文件名方案的项目，保持原代码不变即bundle文件不更改，重新发布资源链接为更改，能够走缓存。runtime文件每次build都会改变，单独拎出，重新构建发布只影响该资源的请求。不过要注意，如果不是spa，多资源入口，每个都拆出单独的runtime，会导致多个重复代码文件被加载。
* **开启treeshaking**，不过treeshaking要求有点多，很多资源包不符合规范用不了
    * 使用 ES2015 模块语法（即 import 和 export）。所以如果npm包里指定了`module`的入口（这个入口是es module源码文件），webpack会优先使用这个入口的文件，且关闭babel之类工具将 ES2015 模块语法转换为 CommonJS 模块的处理，babel配置`modules:false`
    * 有副作用不能被treeshaking的文件，如polyfill相关的，在项目 package.json 文件中，添加一个 `sideEffects` 属性，指明对应文件。
    * 引入一个能够删除未引用代码(dead code)的压缩工具(minifier)（例如 UglifyJSPlugin），这个倒是production模式下webpack默认配置就有。毕竟treeshaking在webpack层面做的只是标记使用片段，未使用片段的删除是通过UglifyJS去做的。
    这个最大的尴尬点是第三方包，有些没有直接提供`module`入口，有些没有指明sideEffects，虽然webpack允许我们在loader层去配置这个，但是我们也不能确定那么多npm包，具体哪个有副作用，这个使用姿势不现实。这个还是需要时间成为社区一种规范做法的。
* **使用`mini-css-extract-plugin 或者extract-text-webpack-plugin`，将css单独生成一个文件**
* polyfile文件单独拆离，根据浏览器环境动态加载：[参考https://webpack.docschina.org/guides/shimming/#%E5%8A%A0%E8%BD%BD-polyfills](https://webpack.docschina.org/guides/shimming/#%E5%8A%A0%E8%BD%BD-polyfills)
* `optimization.minimize: true`（pro默认状态）

# 两种模式下公用的优化方案

- 使用了`ant-design`，务必使用`babel-plugin-import`插件来按需加载模块，既能减少构建体积，也减少了开发环境下的构建代码处理量，提高了构件速度
- **异步动态import**，对构建速度的提升作用类似拆项目，对构建结果更是减少了单个资源的体积
- **ignore某些用不到的大资源**，譬如moment里的locale相关文件，用分析工具会看到体积巨大，可以配置`new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)`


希望以上内容能实践下来真的帮你提高工作中webpack的速度，而不仅是作为面试的回答。

# 参考

[使用webpack4提升编译速度](https://juejin.im/entry/5c302140f265da611b587f99)
[加速 Webpack](https://www.ibm.com/developerworks/cn/web/wa-lo-expedite-webpack/index.html)
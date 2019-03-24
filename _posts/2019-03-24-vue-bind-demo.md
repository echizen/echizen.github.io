---
layout: post
title: "vue-双向绑定的简单实现"
description: "vue-双向绑定的简单实现"
category: tech
tags: ['vue']
---
{% include JB/setup %}

了解了上一篇文章中介绍的内容后，应该会清楚vue中一个重要功能`v-model`的实现了。

先上simple版代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Two-way data-binding</title>
</head>
<body>
    <div id="app">
        <input type="text" v-model="text">
        {{ text }}
    </div>
    <script>
        function observe (obj, vm) {
            Object.keys(obj).forEach(function (key) {
                defineReactive(vm, key, obj[key]);
            });
        }
        function defineReactive (obj, key, val) {
            var dep = new Dep();
            Object.defineProperty(obj, key, {
                get: function () {
                    if (Dep.target) dep.addSub(Dep.target);
                    return val
                },
                set: function (newVal) {
                    if (newVal === val) return
                    val = newVal;
                    dep.notify();
                }
            });
        }
        function nodeToFragment (node, vm) {
            var flag = document.createDocumentFragment();
            var child;
            while (child = node.firstChild) {
                compile(child, vm);
                flag.appendChild(child);
            }
            return flag;
        }
        function compile (node, vm) {
            var reg = /\{\{(.*)\}\}/;
            // 节点类型为元素
            if (node.nodeType === 1) {
                var attr = node.attributes;
                // 解析属性
                for (var i = 0; i < attr.length; i++) {
                    if (attr[i].nodeName == 'v-model') {
                        var name = attr[i].nodeValue; // 获取v-model绑定的属性名
                        node.addEventListener('input', function (e) {
                            // 给相应的data属性赋值，进而触发该属性的set方法
                            vm[name] = e.target.value;
                        });
                        node.value = vm[name]; // 将data的值赋给该node
                        node.removeAttribute('v-model');
                    }
                }
                new Watcher(vm, node, name, 'input');
            }
            // 节点类型为text
            if (node.nodeType === 3) {
                if (reg.test(node.nodeValue)) {
                    var name = RegExp.$1; // 获取匹配到的字符串
                    name = name.trim();
                    new Watcher(vm, node, name, 'text');
                }
            }
        }
    
        function Watcher (vm, node, name, nodeType) {
        //  this为watcher函数
            Dep.target = this;
        //  console.log(this);
            this.name = name;
            this.node = node;
            this.vm = vm;
            this.nodeType = nodeType;
            this.update();
            Dep.target = null;
        }
        Watcher.prototype = {
            update: function () {
                this.get();
                if (this.nodeType == 'text') {
                    this.node.nodeValue = this.value;
                }
                if (this.nodeType == 'input') {
                    this.node.value = this.value;
                }
            },
            // 获取data中的属性值
            get: function () {
                this.value = this.vm[this.name]; // 触发相应属性的get
            }
        }
        function Dep () {
            this.subs = []
        }
        Dep.prototype = {
            addSub: function(sub) {
                this.subs.push(sub);
            },
            notify: function() {
                this.subs.forEach(function(sub) {
                    sub.update();
                });
            }
        };
        function Vue (options) {
            this.data = options.data;
            var data = this.data;
            observe(data, this);
            var id = options.el;
            var dom = nodeToFragment(document.getElementById(id), this);
            // 编译完成后，将dom返回到app中
            document.getElementById(id).appendChild(dom);
        }
        var vm = new Vue({
            el: 'app',
            data: {
                text: 'hello world'
            }
        });

        setTimeout(()=>{
          vm.text = 'fuck world'
        }, 1000)
    </script>
</body>
</html>
```

简单版的`defineReactive`、`Dep`、`Watcher`加持后，响应式数据部分完成了，这个流程是数据-视图的流动，保证了通过`vm.text=`调用数据更新时，触发`Watcher`里的`update`方法更新相应节点dom内容；而视图-数据的反应，则是通过事件绑定来完成的，初次渲染视图时，就对视图里`v-model`的节点做了事件监听，保证`input`事件触发时，能将用户输入的数据及时反映到数据层`vm`的对应data上。

流程图展示就是：

![](https://upload-images.jianshu.io/upload_images/3360875-4428cd8a2fdc4fac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/730/format/webp)

这样就是一个双向绑定的简单实现了，当然这里只为了展现这个核心原理，vue中的真实处理要比这精细的多，不会有各种判断的硬编码，兼容各种节点类型各种情况。更新视图也没有这么简单直接调用`this.node.nodeValue`或者`this.node.value`直接赋值，而是有个批量`patch`的过程，将一个事件循环周期内需要更新的内容先收集起来，再一次性更新来提高性能。不过看完后，是否有了双向绑定我也能实现的感觉呢，并不是多么神密的设计哦。
---
layout: post
title: "for in 循环js对象的顺序"
description: "for in 循环js对象的顺序，js 对象无序，Map有序"
category: tech
tags: [js]
---
{% include JB/setup %}

猜猜以下内容的结果

    var obj = {
      city: "Beijing",
      12: 12,
      7: 7,
      0: 0,
      "-2": -2,
      "age": 15,
      "-3.5": -3.5,
      7.7: 7.7,
      _: "___",
      online: true,
      3: 3,
      "23": "23",
      "44": 44
    }

    for(key in obj){
      console.log(key)
    }
    
- chrome
    ![http://s2.mogucdn.com/p2/170312/114994688_3ah91hefjk43la5abl4979gg6bc9g_400x1008.png](http://s2.mogucdn.com/p2/170312/114994688_3ah91hefjk43la5abl4979gg6bc9g_400x1008.png)

- safari
    ![http://s2.mogucdn.com/p2/170312/114994688_141ih5lj2hja279cgk7hhd0i4i116_388x1092.png](http://s2.mogucdn.com/p2/170312/114994688_141ih5lj2hja279cgk7hhd0i4i116_388x1092.png)

- firefox
    ![http://s2.mogucdn.com/p2/170312/114994688_6akjbaej9g3bc33da6fledljha18l_1224x596.png](http://s2.mogucdn.com/p2/170312/114994688_6akjbaej9g3bc33da6fledljha18l_1224x596.png)

- IE

    ![http://s2.mogucdn.com/p2/170312/114994688_4jh5f28f2ef5e15kehi3585j1j8he_336x1022.png](http://s2.mogucdn.com/p2/170312/114994688_4jh5f28f2ef5e15kehi3585j1j8he_336x1022.png)

## 标准

- 根据 ECMA-262（ECMAScript）第三版中描述，for-in 语句的属性遍历的顺序是**由对象定义时属性的书写顺序决定的**。[ECMA-262 3rd Edition](http://www.ecma-international.org/publications/files/ECMA-ST-ARCH/ECMA-262,%203rd%20edition,%20December%201999.pdf)【参考12.6.4 The for-in Statement。】
   ![http://s17.mogucdn.com/p1/170312/idid_ifqwkm3fmuytkyjzmuzdambqgyyde_1980x1392.png](http://s17.mogucdn.com/p1/170312/idid_ifqwkm3fmuytkyjzmuzdambqgyyde_1980x1392.png)

- ECMA-262（ECMAScript）第五版规范中，对 for-in 语句的遍历机制又做了调整，**属性遍历的顺序是没有被规定的**。[ECMA-262 5rd Edition](http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf)
【参考13.7.5 The for‑in and for‑of Statements】

    ![http://s16.mogucdn.com/p1/170312/idid_ifqtknrqgmytkyjzmuzdambqgyyde_2004x1406.png](http://s16.mogucdn.com/p1/170312/idid_ifqtknrqgmytkyjzmuzdambqgyyde_2004x1406.png)

## 各大浏览器的行为

- Chrome、 sarari、 firebox、 Opera 中使用 for-in     语句遍历对象属性时会遵循一个规律，它们会先提取所有 key 的 parseFloat 值为**非负整数**的属性， 然后**根据数字顺序**对属性排序首先遍历出来，然后按照**对象定义的顺序遍历余下的所有属性**。order is preserved with the major exception of keys like "7" that parse as integers and are handled differently by Chrome/V8

- 其它浏览器则完全按照对象定义的顺序遍历属性。

Chrome等现代浏览器较新版本 的 JavaScript 解析引擎遵循的是新版 ECMA-262 第五版规范（看到11年stackoverflow上的讨论）。因此，使用 for-in 语句遍历对象属性时遍历书序并非属性构建顺序。而 IE6 IE7 IE8 Firefox Safari 的 JavaScript 解析引擎遵循的是较老的 ECMA-262 第三版规范，属性遍历顺序由属性构建的顺序决定。

## 有顺序的遍历

> 4.3.3 Object
    An object is a member of the type Object. It is an unordered collection of properties each of which contains a primitive value, object, or function. A function stored in a property of an object is called a method. 

for-in、object.keys(objectX).map 语句无法保证对象的遍历顺序，应尽量避免编写依赖对象属性顺序的代码。

- 使用数组
- 使用es6 的Map类型:
    > A Map iterates its elements in insertion order, whereas iteration order is not specified for Objects.


譬如之前的例子：

    var myMap = new Map();
    myMap.set("city", "Beijing");
    myMap.set(12, 12);
    myMap.set(7, 7);
    myMap.set(0, 0);
    myMap.set("-2", -2);
    myMap.set("age", 15);
    myMap.set("-3.5", -3.5);
    myMap.set(7.7, 7.7);
    myMap.set("_", "___");
    myMap.set("online", true);
    myMap.set(3, 3);
    myMap.set("23", "23");
    myMap.set("44", 44);
    //不支持for in 遍历
    for (var [key, value] of myMap) {
      console.log(key + ' = ' + value);
    }
    
![http://s2.mogucdn.com/p2/170312/114994688_44h919l168kg3d4gfc6k4ff138gk9_548x942.png](http://s2.mogucdn.com/p2/170312/114994688_44h919l168kg3d4gfc6k4ff138gk9_548x942.png)
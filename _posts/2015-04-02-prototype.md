---
layout: post
title: "初识原型和构造器"
description: "js原型、构造器、继承"
category: tech
tags: [js, pro]
---
{% include JB/setup %}


首先，阐述一下，为了弄清楚原型、闭包，我看了很多博客文章，最终真正有感悟的是这篇：

[http://dmitrysoshnikov.com/ecmascript/javascript-the-core/](http://dmitrysoshnikov.com/ecmascript/javascript-the-core/)

感谢原作者阐述的这么详细。

首先解释一下，ECMAScript是一套规范，javascript是实现这套规范的语言。

**原型**

一个对象的prototype属性在chrome控制台下可以在__proto__里看到，是以内部的[[Prototype]]属性来引用的。

这篇文章整理的最精彩的是这些图片，清晰的阐述了逻辑。

	var foo = {
  		x: 10,
  		y: 20
	};

![image](https://echizen.github.io/assets/blog-img/QQ20150412-1@2x.png)

可以看到定义了一个对象后，他除了拥有自己显式定义的属性和方法外，还拥有隐式的__proto__属性,指向他的原型prototype属性。

**原型链**

    var a = {
      x: 10,
      calculate: function (z) {
        return this.x + this.y + z
      }
    };

    var b = {
      y: 20,
      __proto__: a
    };

    var c = {
      y: 30,
      __proto__: a
    };

    // call the inherited method
    b.calculate(30); // 60
    c.calculate(40); // 80
    
![image](https://echizen.github.io/assets/blog-img/QQ20150412-2@2x.png)

如果一个属性或者一个方法在对象自身中无法找到，然后它会尝试在原型链中寻找这个属性/方法。如果这个属性在原型中没有查找到，那么将会查找这个原型的原型，以此类推，遍历整个原型链（当然这在类继承中也是一样的）。第一个被查找到的同名属性/方法会被使用。因此，一个被查找到的属性叫作继承属性。如果在遍历了整个原型链之后还是没有查找到这个属性的话，返回undefined值。

注意，继承方法中所使用的this的值被设置为原始对象，而并不是在其中查找到这个方法的（原型）对象。也就是，在上面的例子中this.y取的是b和c中的值，而不是a中的值。但是，this.x是取的是a中的值，并且又一次通过原型链机制完成。

如果没有明确为一个对象指定原型，那么它将会使用__proto__的默认值－Object.prototype。Object.prototype对象自身也有一个__proto__属性，这是原型链的终点并且值为null。

也就是说通过原型指定的方式建立起的一群对象会形成一个原型链，支部的对象能访问根部对象的属性，这个属性的访问是通过原型链的向上查找定位到的。根部对象的原型指向Object,Object的原型指向null。

##构造函数

构造函数自动地为新创建的对象设置一个原型对象。这个原型对象存储在ConstructorFunction.prototype属性中。

我们可以使用构造函数来重写上一个拥有对象b和对象c的例子。因此，对象a（一个原型对象）的角色由Foo.prototype来扮演：

    // a constructor function
    function Foo(y) {
      // which may create objects
      // by specified pattern: they have after
      // creation own "y" property
      this.y = y;
    }

    // also "Foo.prototype" stores reference
    // to the prototype of newly created objects,
    // so we may use it to define shared/inherited
    // properties or methods, so the same as in
    // previous example we have:

    // inherited property "x"
    Foo.prototype.x = 10;

    // and inherited method "calculate"
    Foo.prototype.calculate = function (z) {
      return this.x + this.y + z;
    };

    // now create our "b" and "c"
    // objects using "pattern" Foo
    var b = new Foo(20);
    var c = new Foo(30);

    // call the inherited method
    b.calculate(30); // 60
    c.calculate(40); // 80

    // let's show that we reference
    // properties we expect

    console.log(

      b.__proto__ === Foo.prototype, // true
      c.__proto__ === Foo.prototype, // true

      // also "Foo.prototype" automatically creates
      // a special property "constructor", which is a
      // reference to the constructor function itself;
      // instances "b" and "c" may found it via
      // delegation and use to check their constructor

      b.constructor === Foo, // true
      c.constructor === Foo, // true
      Foo.prototype.constructor === Foo // true

      b.calculate === b.__proto__.calculate, // true
      b.__proto__.calculate === Foo.prototype.calculate // true

    );
    
![image](https://echizen.github.io/assets/blog-img/QQ20150412-3@2x.png)

 1. js中的一个函数在创建的时候，会自动被分配到一个prototype属性，这个属性是指向自己本身的一个对象。拥有自己本身拥有的所有属性（上面的示例中Foo.prototype.y存在）。
 2. 函数的prototype属性又有一个constructor属性，这个属性指向函数本身。
 3. 我们可以通过拓展函数的prototype属性，让它的实例（继承自他的函数）拥有这些属性。（这时候，该函数本身并不带有这些属性，其实这个设计模式就是要该函数作为公共函数，并不需要这些私有、留给实例拓展的属性。）
 
 
##混淆点

最开始看这个时很纳闷__proto__和propotype的关系的.

 1. 对于对象来说，__proto__是对象原型prototype的引用。在chrome控制台下，是无法直接访问对象propotype属性的，只能访问__proto__属性，打印出来是对象本身。
 2. 函数的__proto__指向空函数function Empty()
，函数的prototype属性指向函数的同名的空对象。在chrome控制台下可访问函数的__proto__属性和prototype属性。 

还是之前的例子：

	var a = {
	  x: 10,
	  calculate: function (z) {
	    return this.x + this.y + z
	  }
	};
	
	a.__proto__;  // Object {}
	a.property;   // undefined 
	
=============

	function Foo(y) {
	  this.y = y;
	}
	Foo.__proto__; // function Empty() {}
	Foo.prototype; //Foo {}
	
	Foo.prototype.y; //undefined
	Foo.prototype.constructor(1);
	Foo.prototype.y; // 1
	
##困惑

对于上面为什么调用了构造函数之后，就可以通过原型访问函数内部变量我表示不理解。

============

new一个已存在的函数会继承他的原型，拥有他的属性。

##prototype是什么

开始我看了很多实例，越看越郁闷prototype是什么。“原型是一个对象，其他对象可以通过它实现属性继承。”

这么说吧，你定义了一个函数,并且对他进行了实例化：

	var baseObj = function(){};
	var extendObj = new baseObj();
	
这时，extendObj拥有了所有baseObj的属性和方法，并且通过prototype属性可以拓展。baseObj称为extendObj的构造器，prototype称为原型。extendObj称为对象。

 **通过`baseObj.prototype.functionA`拓展的属性和方法只对实例化后的对象存在，baseObj本身不带有functionA.且再次实例化baseObj，baseObj的对象仍带有属性functionA**
 
 通过这个特征，那么使用原型拓展，而不是直接拓展对象的属性的作用是，拓展了原型，就一次性拓展了所有原型的对象。这在很多情况下都是很有用的。
 
##测试代码

	var baseObj = function(){
        this.name = 'echizen';
        this.getName = function() {
            console.log('the name in baseObj:'+this.name);
        };
    };

    var extendObj = new baseObj();

    //对象可以继承原型的属性
    extendObj.getName();
    //console: the name in baseObj:echizen

    //通过prototype原型的改变会体现在对象上
    baseObj.prototype.age = 0;
    extendObj.age;
    //console:0

    //对象的改变不影响原型
    extendObj.age = 1;
    extendObj.age;
    //console:1
    baseObj.prototype.age;
    //console:0
    baseObj.age;
    //undefined

    baseObj.sex = '女';
    extendObj.sex;
    //undefined

    var extendAnother = new baseObj();
    extendAnother;

最后的这个结果特别有代表性：


1. **原型扩展的属性可被所有对象共享，不管这个对象是先于原型的这个属性创建还是晚于：**extendAnother已拥有所有`baseObj`及其所拓展的属性。
2. **构造器原型（prototype）的控制器（constructor）指向构造器本身**：函数的constructor指向的是函数的引用。这里可以看到，`baseObj.prototype.constructor`指向了baseObj本身，并且拥有我们后来给baseObj赋的属性sex.
3. **每个新创建的函数的prototype都指向function Empty() {}**:`baseObj.constructor.prototype`指向了function Empty() {}
4. **函数的内部原型__proto__指向其构造器的原型constructor.prototype**:所以baseObj.__proto__== baseObj.constructor.prototype; extendObj.__proto__==extendObj.constructor.prototype。
5. **所有新创建的构造器/函数的__proto__都指向（Empty function），他是Function.prototype**


函数的内部原型(__proto__),构造器的原型（prototype）。


##作用
其实我看一个东西最关注的是他能干什么，如果不是最近笔试遇到这个，我估计不会关注这个。

现在在我看来，原型的存在是为了对象的继承和属性扩展，毕竟js不是面向对象的语言，没有类的概念。利用原型我们可以是开发模块化，把共同的部分抽出来作为一个对象，再来拓展这个对象完成功能多样化。

作用就是我们可以将js内置的一些对象函数实例化，然后扩展他们，这样我们就能在已有的东西上做开发，就能使用内置的一些函数对象的属性和方法了。

参考：
[http://www.cnblogs.com/snandy/archive/2012/09/01/2664134.html](http://www.cnblogs.com/snandy/archive/2012/09/01/2664134.html)

[http://blog.csdn.net/hikvision_java_gyh/article/details/8937157
](http://blog.csdn.net/hikvision_java_gyh/article/details/8937157
)

[http://weizhifeng.net/javascript-the-core.html](http://weizhifeng.net/javascript-the-core.html)

[https://bonsaiden.github.io/JavaScript-Garden/zh/](https://bonsaiden.github.io/JavaScript-Garden/zh/)

	

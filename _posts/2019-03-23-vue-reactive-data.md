---
layout: post
title: "vue源码-响应式数据原理"
description: "vue响应式数据的设计，vue双向绑定实现"
category: tech
tags: ['vue']
---
{% include JB/setup %}

开始写vue源码阅读后的总结

说到vue的设计，不得不提响应式数据的实现，这是vue的一个鲜明特点了。硬文预告。

官网的图很好的解释了响应式的设计：

![image](https://cn.vuejs.org/images/data.png)

通过给视图中使用到的data和props设置`Object.defineProperty` 的`getter`和`setter`方法，在getter中进行依赖收集，只收集视图render依赖的数据。data有一个Deps订阅者，在修改的时候会通知所有的Watcher观察者
setter中则去触发观察者Watcher的notify，进行后续的virtual dom比对和视图层更新。

# 源码

这部分相关的代码在`src/core/observer`中。关键片段：

## defineReactive

> Define a reactive property on an Object.

设置getter和setter，将操作对象变成响应式。

```javascript
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          dependArray(value)
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

`defineReactive`用在props、data等数据的每个属性上。可以看出对于每个数据都初始化了一个依赖收集器`dep = new Dep()`，然后在`get`方法里调用`dep.depend()`来进行依赖收集，在`set`方法中调用`dep.notify`通知依赖收集器发生变更，进行相应的处理。

与之相关的还有以下两部分：

- observe(): 是props、data等数据的每个属性上直接调用的方法，也就是调用源头。一波判断后会调用observer。通过hasOwn(value, '__ob__')来判断是否已经存在一个observer了，存在则直接返回

- Observer{}: Observer class that are attached to each observed object. Once attached, the observer converts target object's property keys into getter/setters that collect dependencies and dispatches updates.
  为数据加上响应式属性进行双向绑定, 对 对象和数组分别进行defineReactive操作。如果是数组则对数组的每一项进行observe；若是对象，则在defineReactive方法的`let childOb = !shallow && observe(val)` 这步对每个属性都做了observe的操作

# Dep

> A dep is an observable that can have multiple directives subscribing to it.

```javascript
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
Dep.target = null
const targetStack = []

export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

export function popTarget () {
  Dep.target = targetStack.pop()
}
```

Dep是发布者，也是依赖收集者。

可以订阅多个观察者，依赖收集之后Deps中会存在一个或多个Watcher对象，在数据变更的时候通知所有的Watcher。
`subs`记录订阅者列表，有用`addSub`, `removeSub`增删方法, `depend()`则是`Dep.target.addDep(this)` ,`notify`是遍历 `subs[i].update()`

`Dep.target`这个文件作用域变量比较难懂，他其实是观察者Watcher。`Watcher.get`方法调用了`pushTarget`，将Watcher设置为了`Dep.target`。所以Dep和Watcher的关系非常鬼畜，`depIns.depend`里是在向`WatcherIns.addDep(depIns)`，向watcher的`newDeps`队列里添加`depIns`, `WatcherIns.addDep`又会调用`dep.addSub(watchIns)`，向dep的`subs`队列里添加watcher，所以是在进行相互关联。**Dep是个中间枢纽，联系着源数据和观察者，并且做到一个Dep联系多个观察者**

## watcher

> A watcher parses an expression, collects dependencies, and fires callback when the expression value changes. This is used for both the $watch() api and directives.

```javascript
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: ISet;
  newDepIds: ISet;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ) {
    this.vm = vm
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` +
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /**
   * Evaluate the getter, and re-collect dependencies.
   */
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      if (this.user) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      } else {
        throw e
      }
    } finally {
      // "touch" every property so they are all tracked as
      // dependencies for deep watching
      if (this.deep) {
        traverse(value)
      }
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  /**
   * Add a dependency to this directive.
   */
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  /**
   * Clean up for dependency collection.
   */
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  /**
   * Subscriber interface.
   * Will be called when a dependency changes.
   */
  update () {
    /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }

  /**
   * Scheduler job interface.
   * Will be called by the scheduler.
   */
  run () {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }

  /**
   * Evaluate the value of the watcher.
   * This only gets called for lazy watchers.
   */
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  /**
   * Depend on all deps collected by this watcher.
   */
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  /**
   * Remove self from all dependencies' subscriber list.
   */
  teardown () {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so we skip it
      // if the vm is being destroyed.
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this)
      }
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      this.active = false
    }
  }
}

```

可以看出，watcher和Dep建立了关联，通过Dep.notify-> subs.update = watcher.update - > watcher.run - > cb ，这个流程确保在监听数据发生变更后调用对应的回调处理函数

# 数据更新到视图更新流程

```
修改数据
触发set
Dep.notify
watcher.update
watcher.run
    watcher.get - getter：这里取名很混淆，这个getter不是字面意思的getter也是不是expOrFn的字面意思，而是真正监听到数据变更后要做出响应操作的函数，譬如更新视图
        watcher建立时的expOrFn
            更新dom或其他类型的响应操作
```

再看看更新视图的部分：

![](https://s10.mogucdn.com/mlcdn/c45406/190323_3bc9ha0a0i4cc667470c82b0ljhgk_946x1242.png)

所以当某个响应式数据发生变化的时候，它的setter函数会通知闭包中的Dep，Dep则会调用它管理的所有Watch对象。触发Watch对象的update实现。异步执行update的时候，会调用queueWatcher函数。
Watch对象并不是立即更新视图，而是被push进了一个队列queue，此时状态处于waiting的状态，这时候继续会有Watch对象被push进这个队列queue，等到下一个tick运行时，这些Watch对象才会被遍历取出，更新视图。`nextTick(flushSchedulerQueue)`中的flushSchedulerQueue是下一个tick时的回调函数，主要目的是执行Watcher的run函数，用来更新视图

# 灵魂发问

你理解响应式数据了么？如果觉得理解了，不如问问自己：有哪些watcher呢？

一开始读这个的我以为视图层依赖的每个数据都对应一个watcher，后来被同事反问啪啪打脸觉得自己蠢，watcher对应的回调是视图更新函数，所以怎么可能data的每个属性都对应着一个watcher实例，那多个data的属性变化，就会调用多次视图更新，显然不合理。

我总结出有这么几种watcher（不一定全面）：

- render watcher：就是文中举例的视图层依赖数据的watcher，当视图层依赖的某个数据变更时，调用对应视图的更新。
- v-model watcher：双向绑定型的watcher，框架自动绑定的watcher，保证视图变化通过事件绑定调用触发对应的数据变化，数据变化触发对应的视图部分更新
- computed\watch watcher：手动添加的watcher，就是每个`computed: {}`和`watch: {}`的每一项对应着一个watcher实例，他的回调是要执行对应的用户定义函数，完成相应数据的被通知变更。

# 参考

首推官方文档：[深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html)

ps:还有人在知乎喷vue的中文文档不好，一个框架官网连核心内容的原理都介绍了，简直良心。

answershuto的文章是我阅读vue源码的指导性文章，感恩，写的非常好~

[Vue.js 源码解析](https://github.com/answershuto/learnVue)
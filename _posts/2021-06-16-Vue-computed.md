---
layout: post
title: 'Vue实例化执行顺序'
date: 2021-06-16 13:44:49
author: 'Youzi'
catalog: true
tags:
  - JS
  - Vue
  - 面试
---

# Vue实例化执行顺序

响应式系统核心的三大类：

1. `observe`：遍历 `data` 中的属性，使用 `Object.defineProperty` 的 `get/set` 方法对其进行数据劫持；
2. `dep`：每个属性拥有自己的消息订阅器 `dep`，用于存放所有订阅了该属性的观察者对象；
3. `watcher`：观察者（对象），通过 `dep` 实现对响应属性的监听，监听到结果后，主动触发自己的回调进行响应。

1. `initProps`
2. `initMethods`
3. `initData`
4. `initComputed`
5. `initWatch`

## `initComputed`

遍历`computed`对象，实例化每一个计算属性`Watcher`，

```js
// key是computed对象的key
// watchers是存放所有计算属性订阅者实例的对象
watchers[key] = new Watcher(
  vm, // 组件实例
  getter || noop, // get函数
  noop,
  { lazy: true } // 标记该watcher是一个计算属性的watcher
)
```

实例化后，调用`defineComputed`，`userDef`是一个函数（或是有`get | set`两个方法的对象）

```js
defineComputed(vm, key, userDef)
```

`Watcher`的构造函数，`watcher`对象是一个观察者，通过实例化`dep`，

```js
class Watcher {
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    if (options) {
      this.lazy = !!options.lazy
    } 
    if (this.computed) {
      this.value = undefined
      // 执行这一句
      this.dep = new Dep()
    } else {
      this.value = this.get()
    }
  }
  // get函数，调用watcher实例的get函数并得到返回值
  get () {
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      value = this.getter.call(vm, vm)
    } catch (e) {
      
    } finally {
      popTarget()
    }
    return value
  }
  // 
  update () {
    if (this.computed) {
      if (this.dep.subs.length === 0) {
        this.dirty = true
      } else {
        this.getAndInvoke(() => {
          this.dep.notify()
        })
      }
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }
 
  evaluate () {
    if (this.dirty) {
      this.value = this.get()
      this.dirty = false
    }
    return this.value
  }
 
  depend () {
    if (this.dep && Dep.target) {
      this.dep.depend()
    }
  
}
```

`Dep`的构造函数

```js
export default class Dep {
  static target: ?Watcher;
  subs: Array<Watcher>;
 
  constructor () {
    this.id = uid++
    this.subs = []
  }
 
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
 
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
 
  notify () {
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
 
Dep.target = null
```
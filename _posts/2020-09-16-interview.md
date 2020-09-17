---
layout: post
title: '面试总结'
date: 2020-9-16 12:05:29
author: 'Youzi'
catalog: true
tags:
  - JavaScript
  - 面试
---

# 面经

## VueRouter 相关

- 原理
- 两种模式的优劣
- 如何实现页面部分渲染

## Vue3.0 原理

根据回答方向，确定后续问题（compositionAPI、响应式的原理改变等等，待补充）

### 2.0 和 3.0 的区别

1. 重构响应式系统，使用 Proxy 替换 Object.defineProperty，使用 Proxy 优势： •可直接监听数组类型的数据变化 •监听的目标为对象本身，不需要像 Object.defineProperty 一样遍历每个属性，有一定的性能提升 •可拦截 apply、ownKeys、has 等 13 种方法，而 Object.defineProperty 不行 •直接实现对象属性的新增/删除
2. 新增 Composition API，更好的逻辑复用和代码组织
3. 重构 Virtual DOM •模板编译时的优化，将一些静态节点编译成常量 •slot 优化，将 slot 编译为 lazy 函数，将 slot 的渲染的决定权交给子组件 •模板中内联事件的提取并重用（原本每次渲染都重新生成内联函数）
4. 代码结构调整，更便于 Tree shaking，使得体积更小
5. 使用 Typescript 替换 Flow

## 网络原理

- 在浏览器中从 URL 输入到页面展现到底发生什么？
- 浏览器渲染过程
- 渲染中读到`<script> / <style>`会发生什么，`script`标签的阻塞情况，如何避免阻塞

## 事件循环

- 宏任务、微任务，同步任务的执行顺序（同步任务执行完后会 check 异步任务队列）
- requestAnimationFrame 相关

## 防抖节流

## 安全、攻击XSS，CSRF

## HTTPS / HTTP1.X-2.0-3.0

## new做了什么

new 运算接受一个构造器和一组调用参数，实际上做了几件事：

- 以构造器的 prototype 属性（注意与私有字段[[prototype]]的区分）为原型，创建新对象；
- 将 this 和调用参数传给构造器，执行；
- 如果构造器返回的是对象，则返回，否则返回第一步创建的对象。如果构造函数内部返回了一个对象字面量，那这里返回的就应该是构造函数内部的对象，而不是第一步创建的对象。

new 这样的行为，试图让函数对象在语法上跟类变得相似，但是，它客观上提供了两种方式，一是在构造器中添加属性，二是在构造器的 prototype 属性上添加属性。

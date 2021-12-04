---
layout: post
title: '移动端路由切换动画'
date: 2021-03-02
author: 'Youzi'
catalog: true
tags:
  - JS
  - Vue
  - animation
---

# 前言

移动端路由切换，组件切换等动画效果，是很常见的需求，在此记录切换动画的代码；

## 实现思路

在`Vue`项目中，使用`transition`内置组件，可以实现很多动画效果，详见：[https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E8%BF%87%E6%B8%A1class](https://v3.cn.vuejs.org/guide/transitions-enterleave.html#%E8%BF%87%E6%B8%A1class)；

与`vue-router`结合时，文档详见：[https://next.router.vuejs.org/zh/guide/advanced/transitions.html](https://next.router.vuejs.org/zh/guide/advanced/transitions.html)；

以下代码都是基于`vue3.0`版本

```html
<!-- App.vue -->
<router-view v-slot="{ Component, route }">
  <transition :name="route.meta.transitionName">
    <component :is="Component" />
  </transition>
</router-view>
<style>
  .pop-out-enter-active,
  .pop-out-leave-active,
  .pop-in-enter-active,
  .pop-in-leave-active {
    will-change: transform;
    transition: all 300ms;
    height: 100%;
    width: 100%;
    top: 0px;
    position: absolute;
    backface-visibility: hidden;
    perspective: 1000;
  }
  .pop-out-enter-from {
    opacity: 0;
    transform: translate3d(-100%, 0, 0);
  }
  .pop-out-leave-active {
    opacity: 0;
    transform: translate3d(100%, 0, 0);
  }
  .pop-in-enter-from {
    opacity: 0;
    transform: translate3d(100%, 0, 0);
  }
  .pop-in-leave-active {
    opacity: 0;
    transform: translate3d(-100%, 0, 0);
  }
</style>
```

```javascript
const router = createRouter({
  routes,
  history: createWebHashHistory()
});
// 路由的后置钩子
router.afterEach((to, from) => {
  const toDepth = to.meta.index;
  const fromDepth = from.meta.index;
  // 当toDepth < fromDepth时，是退出，一般是向右滑动
  to.meta.transitionName = toDepth < fromDepth ? 'pop-out' : 'pop-in';
  // to.meta.transitionName = toDepth < fromDepth ? 'van-slide-right' : 'van-slide-left';
  console.warn(to.meta.transitionName);
});
```

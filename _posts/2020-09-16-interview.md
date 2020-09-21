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

### 为什么要新增 Composition API，它能解决什么问题

Vue2.0 中，随着功能的增加，组件变得越来越复杂，越来越难维护，而难以维护的根本原因是 Vue 的 API 设计迫使开发者使用 watch，computed，methods 选项组织代码，而不是实际的业务逻辑。另外 Vue2.0 缺少一种较为简洁的低成本的机制来完成逻辑复用，虽然可以 minxis 完成逻辑复用，但是当 mixin 变多的时候，会使得难以找到对应的 data、computed 或者 method 来源于哪个 mixin，使得类型推断难以进行。所以 Composition API 的出现，主要是也是为了解决 Option API 带来的问题，第一个是代码组织问题，Compostion API 可以让开发者根据业务逻辑组织自己的代码，让代码具备更好的可读性和可扩展性，也就是说当下一个开发者接触这一段不是他自己写的代码时，他可以更好的利用代码的组织反推出实际的业务逻辑，或者根据业务逻辑更好的理解代码。第二个是实现代码的逻辑提取与复用，当然 mixin 也可以实现逻辑提取与复用，但是像前面所说的，多个 mixin 作用在同一个组件时，很难看出 property 是来源于哪个 mixin，来源不清楚，另外，多个 mixin 的 property 存在变量命名冲突的风险。而 Composition API 刚好解决了这两个问题。

### Composition API 与 React Hook 很像，说说区别

从 React Hook 的实现角度看，React Hook 是根据 useState 调用的顺序来确定下一次重渲染时的 state 是来源于哪个 useState，所以出现了以下限制不能在循环、条件、嵌套函数中调用 Hook 必须确保总是在你的 React 函数的顶层调用 Hook useEffect、useMemo 等函数必须手动确定依赖关系而 Composition API 是基于 Vue 的响应式系统实现的，与 React Hook 的相比声明在 setup 函数内，一次组件实例化只调用一次 setup，而 React Hook 每次重渲染都需要调用 Hook，使得 React 的 GC 比 Vue 更有压力，性能也相对于 Vue 来说也较慢 Compositon API 的调用不需要顾虑调用顺序，也可以在循环、条件、嵌套函数中使用响应式系统自动实现了依赖收集，进而组件的部分的性能优化由 Vue 内部自己完成，而 React Hook 需要手动传入依赖，而且必须必须保证依赖的顺序，让 useEffect、useMemo 等函数正确的捕获依赖变量，否则会由于依赖不正确使得组件性能下降。虽然 Compositon API 看起来比 React Hook 好用，但是其设计思想也是借鉴 React Hook 的。

### 在 Vue3.0 优雅的使用 v-model

在 Vue2.0 中如何实现双向数据绑定一种是 v-model，另一种是.sync。因为一个组件只能用于一个 v-model，但是有的组件需要有多个可以双向响应的数据，所以就出现了.sync。在 Vue3.0 中为了实现统一，实现了让一个组件可以拥有多个 v-model，同时删除掉了.sync。在 vue3.0 中，v-model 后面需要跟一个 modelValue，即要双向绑定的属性名，Vue3.0 就是通过给不同的 v-model 指定不同的 modelValue 来实现多个 v-model。

参考地址: [https://v3.vuejs.org/guide/migration/v-model.html#overview](https://v3.vuejs.org/guide/migration/v-model.html#overview)

### 2.0 和 3.0 的区别

1. 重构响应式系统，使用 Proxy 替换 Object.defineProperty，使用 Proxy 优势： •可直接监听数组类型的数据变化 •监听的目标为对象本身，不需要像 Object.defineProperty 一样遍历每个属性，有一定的性能提升 •可拦截 apply、ownKeys、has 等 13 种方法，而 Object.defineProperty 不行 •直接实现对象属性的新增/删除
2. 新增 Composition API，更好的逻辑复用和代码组织
3. 重构 Virtual DOM •模板编译时的优化，将一些静态节点编译成常量 •slot 优化，将 slot 编译为 lazy 函数，将 slot 的渲染的决定权交给子组件 •模板中内联事件的提取并重用（原本每次渲染都重新生成内联函数）
4. 代码结构调整，更便于 Tree shaking，使得体积更小
5. 使用 Typescript 替换 Flow

## SSR 服务端渲染相关

### 原理

在客户端请求服务器的时候，服务器到数据库中获取到相关的数据，并且在服务器内部将 Vue 组件渲染成 HTML，并且将数据、HTML 一并返回给客户端，这个在服务器将数据和组件转化为 HTML 的过程，叫做服务端渲染 SSR。而当客户端拿到服务器渲染的 HTML 和数据之后，由于数据已经有了，客户端不需要再一次请求数据，而只需要将数据同步到组件或者 Vuex 内部即可。除了数据之外，HTML 也结构已经有了，客户端在渲染组件的时候，也只需要将 HTML 的 DOM 节点映射到 Virtual DOM 即可，不需要重新创建 DOM 节点，这个将数据和 HTML 同步的过程，又叫做客户端激活。

### 使用 SSR 的好处

有利于 SEO：其实就是有利于爬虫来爬你的页面，因为部分页面爬虫是不支持执行 JavaScript 的，这种不支持执行 JavaScript 的爬虫抓取到的非 SSR 的页面会是一个空的 HTML 页面，而有了 SSR 以后，这些爬虫就可以获取到完整的 HTML 结构的数据，进而收录到搜索引擎中。白屏时间更短：相对于客户端渲染，服务端渲染在浏览器请求 URL 之后已经得到了一个带有数据的 HTML 文本，浏览器只需要解析 HTML，直接构建 DOM 树就可以。而客户端渲染，需要先得到一个空的 HTML 页面，这个时候页面已经进入白屏，之后还需要经过加载并执行 JavaScript、请求后端服务器获取数据、JavaScript 渲染页面几个过程才可以看到最后的页面。特别是在复杂应用中，由于需要加载 JavaScript 脚本，越是复杂的应用，需要加载的 JavaScript 脚本就越多、越大，这会导致应用的首屏加载时间非常长，进而降低了体验感。

## 网络原理

- 在浏览器中从 URL 输入到页面展现到底发生什么？
- 浏览器渲染过程
- 渲染中读到`<script> / <style>`会发生什么，`script`标签的阻塞情况，如何避免阻塞

## 事件循环

- 宏任务、微任务，同步任务的执行顺序（同步任务执行完后会 check 异步任务队列）
- requestAnimationFrame 相关

## window.onload / document.onready / DOMContentLoaded

JS 原生方法`window.onload`和`jQuery(document).ready`对比；

- `ready`比`onload`先触发；因为`ready`只需要`dom`解析完成之后就会触发，而`onload`是在页面所有元素加载完成之后（比如要等待`img, script`等静态资源请求结束后）才会触发。
- `ready`方法可以定义多个，会按序执行；`onload`如果定义多个，只会执行最后一个，毕竟通常写法是给`winodw.onready = function(){}`这样去赋值，会覆盖掉。

另外再补充一下`DOMContentLoaded`事件，这个事件触发机制几乎和`ready`一样。当纯 HTML 被完全加载以及解析时，DOMContentLoaded 事件会被触发，而不必等待样式表，图片或者子框架完成加载。

MDN的一段测试哪个方法先触发的代码，改造了下；

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/1.10.0/jquery.min.js"></script>
  </head>
  <body>
    <div class="controls">
      <button id="reload" type="button">Reload</button>
    </div>

    <div class="event-log">
      <label>Event log:</label>
      <textarea readonly class="event-log-contents" rows="8" cols="30"></textarea>
    </div>
    <img
      src="https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1600663107252&di=86796d61730c55e408c5d6abf44741c9&imgtype=0&src=http%3A%2F%2Fimg1.pclady.com.cn%2Fpclady%2F0711%2F22%2F220923_7%2F1600cat_11002_thumb.jpg"
      alt=""
    />
    <script>
      const log = document.querySelector('.event-log-contents');
      const reload = document.querySelector('#reload');

      reload.addEventListener('click', () => {
        log.textContent = '';
        window.setTimeout(() => {
          window.location.reload(true);
        }, 200);
      });

      window.addEventListener('load', event => {
        log.textContent = log.textContent + 'load\n';
        console.warn('window.onload');
      });
      if (document.readyState === 'loading') {
        // 此时加载尚未完成
        document.addEventListener('DOMContentLoaded', () => {
          console.warn('DOMContentLoaded');
        });
      }
      $(function () {
        console.warn('document.ready');
      });
      document.addEventListener('readystatechange', event => {
        log.textContent = log.textContent + `readystate: ${document.readyState}\n`;
      });

      document.addEventListener('DOMContentLoaded', event => {
        log.textContent = log.textContent + `DOMContentLoaded\n`;
      });
      for (let i = 0; i < 1000000000; i++) {} // 这段同步脚本将会延迟DOM解析,
      // 所以DOMContentLoaded事件将会延迟执行.
    </script>
  </body>
</html>
```

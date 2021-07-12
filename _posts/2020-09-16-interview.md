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

## Vue 原理

根据回答方向，确定后续问题（compositionAPI、响应式的原理改变等等，待补充）

### VDOM diff算法

VDOM中`key`的作用：

- `key`是VDOM对象的标识符，在更新VDOM树的时候起作用；
- 当数据发生变化引起页面渲染时，Vue底层会根据***新的数据**生成**新的VDOM节点**，随后就会进行新的VDOM子树和旧VDOM子树的diff比较，规则如下：
    1. 如果旧的VDOM节点找到了和新VDOM节点相同的`key`，就比较VDOM中的内容：
        1. 如果内容没变，则直接复用之前的真实DOM；
        2. 如果内容变了，则生成新的真实DOM，并替换叼页面中之前的真实DOM
    2. 如果旧节点中没有找到新节点的`key`，则会根据新的VDOM直接创建真实DOM，并渲染到页面中；

- 如果用`index`作为`key`，可能会引发一些问题：
    1. 发生了改变`key`的一些操作，如：数组逆序、乱序等操作，使得旧VDOM的`key`和新VDOM的`key`不一致，会产生虚拟DOM渲染到真实DOM效率的问题（因为只要`key`不一致，就会产生新的真实DOM进行重新的页面渲染，这样就造成了原本可以复用的真实DOM，由于`key`的变化，导致无法复用，产生了渲染效率问题）；
    2. 如果DOM树种还包括了输入类的DOM；此时会将旧的输入类DOM复用，导致页面渲染内容出错的bug；
    3. 如果没有以上的情况，即新数据的`key`没有发生改变，且渲染的组件仅用作展示，不会引发新的真实DOM的更新渲染，就可以用数组的`index`作为`key`。

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

**关于渲染过程的相关问题**，详见这篇[URL 到渲染的过程](/2021/03/08/URL到渲染的过程/)

其他的一些零散问题：

- `get | post`区别；

  - GET 在浏览器回退时是无害的，而 POST 会再次提交请求。
  - GET 产生的 URL 地址可以被 Bookmark，而 POST 不可以。
  - GET 请求会被浏览器主动 cache，而 POST 不会，除非手动设置。
  - GET 请求只能进行 url 编码，而 POST 支持多种编码方式。
  - GET 请求参数会被完整保留在浏览器历史记录里，而 POST 中的参数不会被保留。
  - GET 请求在 URL 中传送的参数是有长度限制的，而 POST 么有。
  - 对参数的数据类型，GET 只接受 ASCII 字符，而 POST 没有限制。
  - GET 比 POST 更不安全，因为参数直接暴露在 URL 上，所以不能用来传递敏感信息。
  - GET 参数通过 URL 传递，POST 放在 Request body 中。

### 跨域问题详解

跨域问题一般有以下几个解决方案：

#### JSONP 模式

利用了`<script>`标签可以请求跨域资源的特性，在 JS 代码中手动构造该标签，且`src`属性改为接口地址（注意只能作用于 GET 请求），参数也可以附带在地址上，甚至回调函数也可以附带在地址上；后端接口接收到请求后，返回约定好的函数名（或者获取前端传过来的函数名），附上真正需要返回的数据做为函数参数，这样前端接收到响应后，就会调用回调函数，执行后续逻辑。

#### CORS

> CORS 模式；跨域资源共享，是一种 HTTP 头的机制，这个机制允许服务器设置除了当前域以外的其他域，访问获得资源；

> 如果使用`fetch`作为请求方法，要提到一个配置`request.mode`，详见：[https://developer.mozilla.org/zh-CN/docs/Web/API/Request/mode#%E5%B1%9E%E6%80%A7%E5%80%BC](https://developer.mozilla.org/zh-CN/docs/Web/API/Request/mode#%E5%B1%9E%E6%80%A7%E5%80%BC)，这个配置里有一些令人迷惑的地方，比如设置了当前请求为`no-cors`，这会让浏览器不去拦截跨域的请求，但是，不能读取任何的`response`对象，这不就是等于没法出去嘛，只是没报错而已。。首先在使用这个模式之前，我们要明确一点，没有配置 CORS 响应头的服务器，请求是没办法突破浏览器的跨域阻拦的，所以首先要在服务器的响应头上设置：

```sh
Access-Control-Allow-Origin: *
```

该字段配置允许通过 CORS 的客户端，`*`表示通配符，任意域都可以通过；也可以配置为一个固定的`URI`，注意不能配置多个`URI`。

仅设置这个字段，CORS 只会允许*简单请求*通过；

> 简单请求：[简单请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS#%E7%AE%80%E5%8D%95%E8%AF%B7%E6%B1%82)

- 特指不会发送`preflight request`（预检请求）的请求，这个后面会讲什么是预检请求；
- 请求方法只能是`GET | HEAD | POST`；
- `Content-Type`字段只能是以下值：

  - `text/plain`
  - `multipart/form-data`
  - `application/x-www-form-urlencoded`

- 其他的要求详见上面的链接，符合字段限制即可；

> 除去简单请求，剩下的的就是非简单请求，这些请求会在发送真实请求之前，先发送一个预检请求：[预检请求](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS#%E9%A2%84%E6%A3%80%E8%AF%B7%E6%B1%82)

预检请求是一个`OPTIONS`方法的请求，目的是为了检测真实请求是否可以被服务器所接受；浏览器会给预检请求附上两个 headers：

- Access-Control-Request-Headers：真实请求携带的自定义头部字段（不属于简单请求的 header）；
- Access-Control-Request-Method：真实请求使用的请求方法；

服务器依靠预检请求，来判断是否通过 CORS，预检请求也会返回这俩字段，表示服务器支持的所有请求头和请求方法，后面会再提到的；预检请求的存在是有必要的，为了应对服务器可能接收的各种奇怪的请求，预先使用预检请求来规避接收到真实请求的风险，对服务器的安全性也更有保障。

> 跨域请求带 cookie

首先在前端处理上，如果用 XHR，则要设置`xhr.withCredentials = true`；如果使用`fetch`要加上`credentials: 'include'`，才会使得请求带上 cookie；另外在后端服务器上，要满足：

- 后端 `Response header` 有 `Access-Control-Allow-Credentials: true`
- 后端 `Response header` 的 `Access-Control-Allow-Origin` 不能是`*`，要明确指定某个域

值得注意的是，**如果后端需要设置 cookie，也需要满足上述条件**

> 跨域请求带自定义头部

跨域请求要带上自定义头部，同样需要后端服务器配合，需要往头部加入`Access-Control-Expose-Headers`字段，字段的值就是需要添加的自定义头部；举个例子，比如前后端自定义了头部`business-http-header`，通过这个头部来传输一些信息，如果不加`Access-Control-Expose-Headers: business-http-header`，则客户端通过响应头是获取不到这个头部携带的信息的，因为并没有暴露给 JS；

> 跨域请求使用除了`GET/POST/HEAD`的其他方法

刚刚说过了简单请求也只包含上述三种方法，所以如果想要发起其他方法的请求比如`PATCH/DELETE`等，还需要后端配置`Access-Control-Allow-Methods`表示支持跨域的其他请求方法；

这个字段和前几个也差不多，就是规定除了简单请求的方法外还允许哪些方法通过 CORS；

> 大量预检请求占用带宽

前面提到过，如果发送的不是简单请求，则浏览器会自动先发送一个预检请求到服务器，获取服务器支持的请求头和请求方法；如果每个非简单请求都这样去做一次预检请求的话，那服务器的 HTTP 连接数会有很多浪费，所以还需要加上`Access-Allow-Max-Age: 100`表示预检请求的返回值，最多能被客户端缓存多长时间（这里写的是 100 秒），一般可以被设置为最大值`86400`秒（即 24 小时）

#### 渲染中读到`<script> / <style>`会发生什么，`script`标签的阻塞情况，如何避免阻塞

## 事件循环

- 宏任务、微任务，同步任务的执行顺序（同步任务执行完后会 check 异步任务队列）
- requestAnimationFrame 相关

## window.onload / document.onready / DOMContentLoaded

JS 原生方法`window.onload`和`jQuery(document).ready`对比；

- `ready`比`onload`先触发；因为`ready`只需要`dom`解析完成之后就会触发，而`onload`是在页面所有元素加载完成之后（比如要等待`img, script`等静态资源请求结束后）才会触发。
- `ready`方法可以定义多个，会按序执行；`onload`如果定义多个，只会执行最后一个，毕竟通常写法是给`winodw.onload = function(){}`这样去赋值，会覆盖掉。

另外再补充一下`DOMContentLoaded`事件，这个事件触发机制几乎和`ready`一样。当纯 HTML 被完全加载以及解析时，DOMContentLoaded 事件会被触发，而不必等待样式表，图片或者子框架完成加载。

MDN 的一段测试哪个方法先触发的代码，改造了下；

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

## 防抖节流

## `this`相关

### 基础

`this`指向最后调用该函数的对象。

#### 绑定规则

- 默认绑定在`window`对象上，但在严格模式下，`this`是指向`undefined`的；
- 隐式绑定在调用对象上；
- 通过`call, apply, bind`方法改变绑定对象；
- 通过`new`，此时`this`指向由`new`构造出来的新对象。

#### `call, apply, bind`

`call / apply`都是立即执行的，一旦被调用，除了改变`this`的指向之外，还会立即执行调用这俩方法的函数，比如`A.call(null)`，会立即执行`A`函数；`bind`不会立即执行而是返回一个新的函数，所以很明显`bind`会在内部新建一个函数；

#### `new` 做了什么

new 运算接受一个构造器和一组调用参数，实际上做了几件事：

- 以构造器的 prototype 属性（注意与私有字段`[[prototype]]`的区分）为原型，创建新对象；
- 将 this 和调用参数传给构造器，执行；
- 如果构造器返回的是对象，则返回，否则返回第一步创建的对象。如果构造函数内部返回了一个对象字面量，那这里返回的就应该是构造函数内部的对象，而不是第一步创建的对象。

new 这样的行为，试图让函数对象在语法上跟类变得相似，但是，它客观上提供了两种方式，一是在构造器中添加属性，二是在构造器的 prototype 属性上添加属性。

#### 优先级顺序

`new > call,apply,bind > 隐式绑定 > 默认绑定`

#### 手写 bind / call / apply

**`bind`**

`bind`函数做了以下几件事，例子代码

```javascript
let newFunc = callFunc.bind(context, args1);
newFunc(args2);
```

- 改变调用函数`newFunc`的`this`指向，指向第一个参数`context`，`args1`可以成为前置参数，在调用`bind`时传入的；
- 返回了一个新函数`newFunc`，在被调用执行时可以后续参数`args2`；
- 特殊情况：如果`newFunc`是一个构造函数（当它被`new`运算符操作时），会忽略第一个参数`context`，将`this`绑定在经过`new`得到的实例对象上；

```javascript
Function.prototype.myBind = function (thisObj, ...args1) {
  // 记录当前调用myBind函数的函数
  let callFunc = this;
  if (typeof callFunc !== 'function') {
    throw new TypeError('call function type error');
  }
  // bind会返回一个新的函数，还记得吧
  // 函数参数是原调用函数的参数
  // 注意newFunc函数始终都要执行的，所以在其内部最后还是要显示调用apply或者call函数
  function newFunc(...args2) {
    // 判断newFunc是否是被new关键字调用的
    if (this instanceof newFunc) {
      // 如果newFunc函数是被new调用的，那此时的this应该指向的是new出来的实例，说白了就是当前的this
      // 在MDN中也提到过，如果调用bind之后返回的函数，用了new运算符，那其实会忽略掉bind的第一个参数thisObj
      return callFunc.call(this, ...args1, ...args2);
    } else {
      // 不是被new调用的情况，那此时的this应该指向myBind第一个参数thisObj
      return callFunc.call(thisObj, ...args1, ...args2);
    }
  }
  // 让新函数的原型指向旧函数的原型，这样可以继承旧函数原型上的所有对象和方法
  newFunc.prototype = callFunc.prototype;
  return newFunc;
};
```

这里的写法，最后一步还有些问题，我们为了让`newFunc`能继承旧函数原型上的方法，手动改变了`prototype`，这就会导致如果修改了`newFunc`的`prototype`，旧函数的原型也会被修改；为了避免，其实也很简单，我们可以利用实例继承的模式，在最后加上：

```javascript
let fn = function () {};
fn.prototype = callFunc.prototype;
newFunc.prototype = new fn();
```

**`call`**

`call`方法做了什么，例子代码：

```javascript
let obj = {
  func1(arg) {
    console.log(arg, this.arg1);
  }
};
let context = {
  arg1: 111
};
obj.func1.call(context, 222);
```

- 将调用函数`func1`执行时的`this`改变为第一个参数`context`，在此基础上，执行`func1`，并返回结果；
- 值得注意的是，`call`方法并不会改变`context`，也不会在`context`上添加`func1`属性；

```javascript
Function.prototype.myCall = function (thisObj, ...args) {
  let callFunc = this;
  thisObj = thisObj || window;
  // 隐式地将this绑定在调用对象上
  thisObj.fn = callFunc;
  let res = thisObj.fn(...args);
  // 调用完后删除该对象上的属性
  Reflect.deleteProperty(thisObj, 'fn');
  return res;
};
```

这样写一般就差不多了，不过有的面试官可能会问如果`thisObj`本身就有`fn`这个属性呢，那其实这里还可以改进一下，就是给`fn`加一个随机数，先判断`thisObj`有没有这个属性，再去调用，最后再删掉；还可以用`Symbol`，独一无二的属性名，保证不会有冲突，而且是私有的，不会被遍历出来，按普通模式去访问也访问不到。

**`apply`**

其实和`call`差不多只是参数要传数组，所以要先判断下传入的参数是否存在，不存在就直接调用，不传参；

## 华为 OD 笔试

### 第一题

全排列问题，具体就是给一个`n, k`，从小到大排序`1~n`的全排列，输出第 k 个值。好棒，第一题我就不会，慌了一下然后跳过了。

### 第二题

字符串处理。`M, G, T`三个单位的转换，第一行是有多少组字符串，给定一组字符串，形如`20G1M3T, 1T2M3G`这样的，比较大小，进制就是 1024，输出从小到大排序后的字符串数组。

**思路：写一个处理单个字符串的函数，转换成整型数然后比较大小；**

> 备注：这个代码通过率 95%，当时也是有时间限制所以没有接着 debug 了

```javascript
let n = Number(readline());
let l = [];
function mgt(str) {
  let sarr = str.split('');
  let sum = 0;
  let tmpS = '';
  for (let el of sarr) {
    /* if (el !== 'M' || el !== 'G' || el !== 'T') {
      tmpS += el;
    } else { */
    switch (el) {
      case 'M':
        sum += Number(tmpS);
        tmpS = '';
        break;
      case 'G':
        sum = sum + 1024 * Number(tmpS);
        tmpS = '';
        break;
      case 'T':
        sum = sum + 1024 * 1024 * Number(tmpS);
        tmpS = '';
        break;
      default:
        tmpS += el;
        break;
      // }
    }
  }
  return sum;
}
while (n--) {
  l.push(readline());
}
l.sort((a, b) => {
  return mgt(a) - mgt(b);
});
for (const i of l) {
  print(i);
}
```

### 第三题

贪心问题。设定场景：硬盘容量 500，输入一串数字字符串`1,100,200,500`表示每个文件的文件大小，问最少需要多少个硬盘才能装下全部文件，文件不能进行拆分之类的操作。

**思路：一开始没想到贪心算法，想着遍历去凑几个文件加起来到 500，后来思考了下，如果把这些文件大小排个序，从小到大，那么我们首先对一个文件大小 x，要找的就是 500-x，如果没找到，就接着用贪心的思想，把前面都没找到的配对的文件组合起来，看看是不是凑到了 500；我也不太会证明这个算法；**

> 备注：这个代码通过率 100%了。

```javascript
let s = readline();
// s = '1,200,100,300';
let arr = s.split(',');
let map = new Map();
arr = arr.map(el => {
  let t = Number(el);
  let i = map.get(t);
  if (i) {
    map.set(t, ++i);
  } else {
    map.set(t, 1);
  }
  return Number(el);
});
arr.sort((a, b) => a - b);
let sum = 0;
let tmpSum = 0;
for (let i = 0; i < arr.length; i++) {
  const a = arr[i];
  let tmpA = map.get(a);
  if (tmpA === 0) {
    // 为0的情况，表示这个数已经被用过了，接着往下就行
    continue;
  }
  if (a === 500) {
    // 等于500就直接继续往下
    sum++;
    map.set(a, --tmpA);
    continue;
  }
  // 标记一下这个值被用过一次了，给它-1
  map.set(a, --tmpA);
  let tmp = map.get(500 - a);
  if (tmp) {
    // 找到了
    sum++;
    map.set(500 - a, --tmp);
    continue;
  }
  // 没找到
  if (tmpSum + a === 500) {
    // 如果前面没配对成功的数字总和，和现在的数字加起来正好是500，就正好也是找到了的情况
    sum++;
    tmpSum = 0;
    // continue;
  } else if (tmpSum + a > 500) {
    // 如果大于500，那表示前面没配对成功的数字总和也不可能再配对了
    // 那就让前面的数字单独占一个硬盘，且让当前数字总和等于现在的这个数字
    sum++;
    tmpSum = a;
    // continue;
  } else {
    // 如果小于500，就接着贪心下一个
    tmpSum += a;
  }
  /* for (let j = i + 1; j < arr.length; j++) {
    const b = arr[j];
  } */
}
if (tmpSum !== 0) {
  // 还需要看最后一个数字是不是被分配了，
  // 如果前面的数字总和是0了,那就表示全部分配完了
  // 如果不是0,表示还要给这些数字再分配一个硬盘
  sum++;
}
print(sum);
// function findTmpSum(n) {}
```

## 零散

1. 虚拟滚动
2. tree-shaking
3. 工程化优势

## Promise 相关内容

### 实现 Sleep 函数

```javascript
const sleep = async time => {
  await new Promise(res => {
    setTimeout(() => {
      // 注意一定要让await语句的状态转变成fulfilled，所以一定要调用res()
      res();
    }, time);
  });
};
const test = async () => {
  for (const i of [1, 2, 3]) {
    await sleep(500);
    console.log(i);
  }
};
```

## 函数实现类

### 科里化

```javascript
// 简单实现，参数只能从右到左传递
function createCurry(func, args) {
  var arity = func.length;
  var args = args || [];
  return function () {
    // let args = Array.from(arguments)
    var _args = [].slice.call(arguments);
    // _args.push(...args)
    [].push.apply(_args, args);
    // 如果参数个数小于最初的func.length，则递归调用，继续收集参数
    if (_args.length < arity) {
      return createCurry.call(this, func, _args);
    }
    // 参数收集完毕，则执行func
    return func.apply(this, _args);
  };
}
```

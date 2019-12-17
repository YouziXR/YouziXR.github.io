# ES6 新特性学习第八篇

## 前言

⑧ 多说辽，这篇是`Proxy`。

### Proxy

`Proxy`字面意思是代理，像是网络请求层面的服务器代理、反向代理一样，是在实际的执行层面架设一层拦截，可以对外部的访问进行过滤和改写。`Vue.js 3.0`就用了这个方法实现了双向数据绑定。

> 写到最后感觉得提出来的一个点：**Proxy 对操作的拦截只针对 Proxy 实例对象上**

### 语法

`let proxy = new Proxy(target, handler)`

`target`表示要代理的目标对象（数组，对象，函数甚至是另一个代理对象）；`handler`是一个对象，其属性是当执行一个操作时定义代理的行为的函数。

其实`Proxy`共支持 13 种针对对象的操作，包括`get / set / has / deleteProperty / ownKeys / getOwnPropertyDescriptor / defineProperty / preventExtensions / getPrototypeOf / isExtensible / setPrototypeOf / apply / construct`：

#### get

`get(target, propKey, receiver)`：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`，参数分别是目标对象，即被代理的对象；属性名；操作针对的对象（一般是 proxy 实例）；

```javascript
let handler = {
  get: function(target, name, receiver) {
    console.log(target === t, name, receiver === proxy);
    return name in target ? target[name] : '?';
  }
};
let t = {};
let proxy = new Proxy(t, handler);
proxy.a = '1'; // true, a, true
proxy.b = null;
console.log(proxy.a, proxy.b); // 1, null
console.log('c' in proxy, proxy.c); // false, '?'
```

上面这个例子对一个对象的`get`方法做了代理，当对象的属性名不存在的时候，默认返回值为`'?'`。

`get`方法可以被继承，在上面代码的基础上：

```javascript
let sub = Object.create(proxy);
sub.x; // '?'
```

#### set

`set(target, propKey, value, receiver)`：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值；多了一个属性值的参数。

```javascript
let proxy = new Proxy(
  {},
  {
    set: (target, key, value) => {
      console.log(target, key, value);
    }
  }
);
proxy.name = '?';
// {} "name" "?"
```

#### apply

`apply(target, object, args)`：拦截 `Proxy` 实例作为函数调用的操作，比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)`；

```javascript
const addFun = (a, b) => a + b;
const anddMultiFun = new Proxy(addFun, {
  apply: (target, obj, args) => {
    console.log(target, obj, args);
    return args[0] * args[1];
  }
});
anddMultiFun(10, 20);
```

#### 无拦截转发

一个比较有意思的点，如果参数`handler`为空对象，即没有设置拦截器，那么会直接转发针对`target`对象的操作，我就搞不懂了为啥要搞两个对象，一个对象是另一个的映射；

### 实例方法

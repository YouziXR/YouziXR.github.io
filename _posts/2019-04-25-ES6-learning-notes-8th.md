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

> **与`writable | configurable`为`false`的状态冲突，会报错**

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

> **与`get`类似，但不会报错，直接不生效**

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

`apply(target, object, args)`：拦截 `Proxy` 实例作为函数调用的操作，比如`proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)、Reflect.apply(...)`，参数为目标对象，目标对象的上下文对象（`this`），参数数组；

```javascript
const addFun = (a, b) => a + b;
const addMultiFun = new Proxy(addFun, {
  apply: (target, obj, args) => {
    console.log(target, obj, args);
    return args[0] * args[1];
  }
});
addMultiFun(10, 20);
addMultiFun.call(null, 10, 20);
addMultiFun.apply(null, [10, 20]);
```

#### has

`has(target, key)`方法拦截判断对象是否具有某个属性，且判断时不区分属性是否在当前对象本身、或者是在对象的原型链上；比如`in`运算符，会被拦截判断，但是不会在`for in`里被判断，换言之在`for in`里不会触发代理；参数为目标对象和属性名；`has`函数返回`true`即代表用`in`运算符访问不到这个属性；

> **如果目标对象不可配置（`configurable: false`），或者不可扩展（`Object.preventExtensions(target)`），会报错**

```javascript
const o = { a: 10 };
let x = Object.create(o);
x.b = 20;
const handler = {
  has(target, key) {
    if (target[key] > 10) {
      console.log(key, '> 10');
      return false;
    } else {
      console.log(key, '<= 10');
      return true;
    }
    // return key in target;
  }
};
let p = new Proxy(x, handler);
'a' in p; // 'a' <= 10
'b' in p; // 'b' > 10
for (const key in p) {
  console.log(key);
}
// b
// a
```

可以看到`for in`没有触发`has`代理，并且不仅在对象自身属性上会触发代理，在原型上的属性也会触发；

#### construct

`construct(target, args, constructor)`用于代理`new`命令；参数为目标对象、构造函数的参数、构造函数；这个代理函数返回值必须是一个对象；

```javascript
var p = new Proxy(function() {}, {
  construct: function(target, args, cons) {
    console.log(target, args, cons);
    return { value: args[0] * 10 };
  }
});

new p(1);
// "called: 1"
// 10
```

#### deleteProperty

`deleteProperty(target, key)`用于代理删除操作；参数为目标对象、属性名；返回布尔值，表示是否成功删除该属性；

> **如果属性不可配置，那么属性不能被删除**

```javascript
const handler = {
  deleteProperty(target, key) {
    if (key === 'a') {
      console.log(target, key);
      delete target[key];
      return true;
    }
  }
};
const o = {
  a: 10
};
let p = new Proxy(o, handler);
delete p.a;
```

#### defineProperty

`defineProperty(target, key, descriptor)`代理`Object.defineProperty`操作；参数为目标对象、属性名、属性描述；如果对象不可扩展，这个方法不能添加对象原本不具有的属性，否则会报错；如果属性不可写或者不可配置，这个方法不能改变这两个设置；

> 原生的`set`方法会触发·defineProperty·的代理方法；

```javascript
let p = new Proxy(
  {},
  {
    defineProperty(t, k, d) {
      console.log(t, k, d);
      return Reflect.defineProperty(t, k, d);
    }
  }
);
p.a = 10;
// {} "a" {value: 10, writable: true, enumerable: true, configurable: true}
```

#### getOwnPropertyDescriptor

`getOwnPropertyDescriptor(target, key)`代理`Object.getOwnPropertyDescriptor()`操作；返回属性描述对象或者`undefined`

#### getPrototypeOf

`getPrototypeOf(target)`用于代理获取对象原型的方法，如`__proto__ | isPrototypeOf | getPrototypeOf | Reflect.getPrototypeOf() | instanceof`；返回原型对象，或者`null`；如果目标对象不可扩展，则必须返回目标对象的原型；

#### isExtensible

`isExtensible(target)`用于代理`Object.isExtensible`操作；返回布尔值，且必须与目标对象的`isExtensible`属性保持一致，事实上只能在这个代理里去改目标对象的属性才能实现更改返回的布尔值了。

#### ownKeys

`ownKeys(target)`用于代理对象自身属性的读取，包括`getOwnPropertyNames | getOwnPropertySymbols | keys | for in`

这个方法会自动过滤三类属性，这三类即使在`ownKeys`方法里显式地返回，也会被过滤掉：

- 不存在在目标对象上的属性；
- 属性名是 symbol 类型的，毕竟只有`getOwnPropertySymbols`才能获取到 symbol 属性名
- `enumerable: false`

> 如果对象不可扩展，那么该方法只能返回目标对象所有属性，且不允许包含多余的属性；

#### preventExtensions

`preventExtensions(target)`代理`Object.preventExtensions()`方法，必须返回一个布尔值；类似`isExtensible`方法，该方法与`Object.isExtensible(proxy)`是相反的值，只有`Object.isExtensible(proxy)`是`false`时，该方法才返回`true`，反之亦然；

所以也只能在这个方法里去调用`Object.preventExtensions(target)`，调用这个方法后可以让`isExtensible`变成`false`；

#### setPrototypeOf

`setPrototypeOf(target, proto)`代理`Object.setPrototypeOf`方法，用于改变对象的原型对象时触发；返回布尔值；如果对象是不可扩展的，那么这个方法不允许使用；

### Proxy.revocable

该方法返回一个可以取消的 Proxy 实例，在调用方法之前，实例都是可用的，调用方法后，再去访问实例会抛出错误；

```javascript
let target = {};
let handler = {};

let { proxy, revoke } = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo; // 123

revoke();
proxy.foo; // TypeError: Revoked
```

### 关于 this

目标对象被代理时，内部的`this`会被指向`proxy`代理的实例，而不会指向原来的目标对象；这会导致一些情况下代理失败；

比如一些原生对象的内部方法，只能通过正确的`this`才能被正确访问，原生对象内部做了一些机制，阻止了非实例对象的访问；例如`Date`对象，调用内部方法`getDate`时，无法通过代理来访问；

```javascript
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);

proxy.getDate();
// TypeError: this is not a Date object.
```

如果想代理原生对象，我们需要在方法内部将`this`对象重新绑定到原生对象的实例对象上，改写上述的代码：

```javascript
const target = new Date();
const handler = {
  get(target, prop) {
    return target[prop].bind(target);
  }
};
const proxy = new Proxy(target, handler);

proxy.getFullYear(); // 2019
```

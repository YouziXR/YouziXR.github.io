# ES6新特性学习第八篇 #

## 前言

⑧多说辽，这篇是`Proxy`。

### Proxy

`Proxy`字面意思是代理，像是网络请求层面的服务器代理、反向代理一样，是在实际的执行层面架设一层拦截，可以对外部的访问进行过滤和改写。`Vue.js 3.0`就用了这个方法实现了双向数据绑定。

### 语法

`let proxy = new Proxy(target, handler)`

`target`表示要代理的目标对象（数组，对象，函数甚至是另一个代理对象）；`handler`是一个对象，其属性是当执行一个操作时定义代理的行为的函数。

几个基础用法的例子：

#### get
`get(target, propKey, receiver)`：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。

    let handler = {
      get: function (target, name) {
        return name in target ? target[name] : '?'
      }
    }
    let proxy = new Proxy({}, handler)
    proxy.a = '1'
    proxy.b = null
    console.log(proxy.a, proxy.b) // 1, null
    console.log('c' in proxy, proxy.c) // false, '?'

上面这个例子对一个对象的`get`方法做了代理，当对象的属性名不存在的时候，默认返回值为`'?'`。

`get`方法可以被继承，在上面代码的基础上：

    let sub = Object.create(proxy)
    sub.x // '?'

#### set


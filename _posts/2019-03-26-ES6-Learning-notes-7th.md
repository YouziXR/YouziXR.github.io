# ES6新特性学习第七篇 #

## 前言 ##

好久没写了，前几天把以前的blog重新看了看，继续学习ES6。

## 第一篇前言 ##

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 原著:阮一峰 ") 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ "阮一峰ES6")

### Promise

Promise是一种异步的解决方案，传统的JS采取了回调函数的模式，这容易造成“回调地狱”的情况。`Promise`对象可以获取异步操作的消息，提供了统一的API，各种异步操作都能用同样的方法进行处理。

`Promise`对象有3中状态：`pending, fulfilled, rejected`，只有异步操作的结果可以决定当前状态是哪种。状态改变只有两种情况：`pengding -> fulfilled, pending -> rejected`，只要这两种情况发生了，状态就固定了，此时称为resolved；状态已经发生改变了，再对`Promise`对象添加回调函数，也会直接得到结果，所以相比较于事件处理，无论在何时都能得到回调的结果。

#### 基本语法

- `Promise`是一个构造函数，所以可以用它来生成实例。构造函数接受一个函数作为参数，MDN将这个函数称为`executor`；
- `executor`函数将两个函数`resolve, reject`作为参数（`executor(resolve, reject)`），两个函数被调用时会将实例的状态改变为`fulfilled || rejected`；
- `executor`内部会执行一些异步操作，执行完后就会调用上面两个函数的其中一个，对状态进行修改。
- 如果`executor`函数抛出错误，状态会变成`rejected`，返回值会被忽略。
- 生成的`Promise`实例，用`then`方法指定`resolved || rejected`状态的处理函数，其中`reject`函数是可选的。

来看个简单但是完整的例子：

    new Promise((resolve, reject) => {
        // do something...
        if (/* success condition */) {
            // 一般异步操作成功的情况会调用resolve，失败调用reject
            resolve(params)
        } else {
            /* failed condition */
            reject(error)
        }
    }).then((params) => {
        // resolve function
    }, (error) => {
        // reject function
    })

再来看一个更具体的例子：

    new Promise((resolve, reject) => {
        let rand = Math.random()
        if (rand > 0.5) {
            setTimeout(() => resolve(`${rand} is greater than 0.5`), rand * 1000 * 3);
        } else {
            setTimeout(() => reject(`${rand} is less than 0.5`), rand * 1000 * 5);
        }
    }).then((result) => {
        console.log(result, 'resolved');
    }).catch((err) => {
        console.log(err, 'rejected');
    });

由于每次调用`then`方法都会返回一个**新的**`Promise`的实例，所以可以使用链式写法在调用了`then`方法后再调用`then`方法；另外`catch`方法是在**抛出异常**或**执行`reject`**时被调用的，其实和`then`方法传入错误处理函数是一样的。

**Promise建立后会立即执行**，`then`指定的回调函数，将在**当前脚本所有同步任务执行完后**才会执行。

    new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    }).then(function () {
        console.log('resolved.');
    });
    console.log('Hi!');
    // Promise
    // Hi!
    // resolved

**`resolve`函数的参数可以是另一个`Promise`的实例**。

    const p1 = new Promise(function (resolve, reject) {
        // ...
    });
    const p2 = new Promise(function (resolve, reject) {
        // ...
        resolve(p1);
    })

这种情况`p1`的状态会传递给`p2`，导致了`p2`状态由`p1`决定，如果`p1`是`pending`，那么`p2`会等待`p1`的状态改变；如果`p1`状态是另外两种状态，那么`p2`的回调函数会立刻执行。

    const p1 = new Promise(function (resolve, reject) {
        setTimeout(() => reject(new Error('fail')), 3000)
    })

    const p2 = new Promise(function (resolve, reject) {
        setTimeout(() => { 
            console.log('???')
            resolve(p1);
        }, 1000)
    })

    p2
    .then(result => console.log(result))
    .catch(error => console.log(error))
    // ???
    // Error: fail

上面代码中，`p1`是一个`Promise`，3秒之后变为`rejected`。`p2`的状态在1秒之后改变，`resolve`方法返回的是`p1`。由于`p2`返回的是另一个 `Promise`，导致`p2`自己的状态无效了，由`p1`的状态决定`p2`的状态。所以，后面的`then`语句都变成针对后者（`p1`）。又过了 2 秒，`p1`变为`rejected`，导致触发`catch`方法指定的回调函数。

另外，调用`resolve || reject`不会终止`executor`函数的执行，但是一般情况下调用了这俩函数之后就应该结束执行了，所以一般在调用这两个函数之后就`return`。


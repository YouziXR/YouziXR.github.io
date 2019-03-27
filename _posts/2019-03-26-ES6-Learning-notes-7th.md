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
- 生成的`Promise`实例，用`then`方法指定`resolved || rejected`

来看个简单的例子：

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


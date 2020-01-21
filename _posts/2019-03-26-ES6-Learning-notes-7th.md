# ES6 新特性学习第七篇

## 前言

好久没写了，前几天把以前的 blog 重新看了看，继续学习 ES6。

## 第一篇前言

本文建立在学习阮一峰老师的 ES6 教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp 'ES6 全套教程 ECMAScript6 原著:阮一峰 ') 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ '阮一峰ES6')

### Promise

Promise 是一种异步的解决方案，传统的 JS 采取了回调函数的模式，这容易造成“回调地狱”的情况。`Promise`对象可以获取异步操作的消息，提供了统一的 API，各种异步操作都能用同样的方法进行处理。

`Promise`对象有 3 中状态：`pending, fulfilled, rejected`，只有异步操作的结果可以决定当前状态是哪种。状态改变只有两种情况：`pending -> fulfilled, pending -> rejected`，**只要这两种情况发生了，状态就固定了**，此时称为 resolved；状态已经发生改变了，再对`Promise`对象添加回调函数，也会直接得到结果，所以相比较于事件处理，无论在何时都能得到回调的结果。

> **`Promise`实例对象的状态只会发生一次改变，当状态确定后就不会再改变了，也就是说在调用了`resolve | reject`函数后，再去调用这两个函数，或者再抛出异常，不会再改变实例的状态**

> 立即执行`resolve`函数的`Promise`对象，在本轮事件循环的结束时进行，而`setTimeout`函数（仅在时间间隔为 0 秒的情况）是在下一轮时间循环开始时执行的；

#### 基本语法

- `Promise`是一个构造函数，所以可以用它来生成实例。构造函数接受一个函数作为参数，MDN 将这个函数称为`executor`；
- `executor`函数将两个函数`resolve, reject`作为参数（`executor(resolve, reject)`），两个函数被调用时会将实例的状态改变为`fulfilled || rejected`；
- `executor`内部会执行一些异步操作，执行完后就会调用上面两个函数的其中一个，对状态进行修改。
- 如果`executor`函数抛出错误，状态会变成`rejected`，返回值会被忽略。
- 生成的`Promise`实例，用`then`方法指定`resolved || rejected`状态的处理函数，其中`reject`函数是可选的。

来看个简单但是完整的例子：

```javascript
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
```

再来看一个更具体的例子：

```javascript
new Promise((resolve, reject) => {
  let rand = Math.random();
  if (rand > 0.5) {
    setTimeout(() => resolve(`${rand} is greater than 0.5`), rand * 1000 * 3);
  } else {
    setTimeout(() => reject(`${rand} is less than 0.5`), rand * 1000 * 5);
  }
})
  .then(result => {
    console.log(result, 'resolved');
  })
  .catch(err => {
    console.log(err, 'rejected');
  });
```

由于每次调用`then`方法都会返回一个**新的**`Promise`的实例，所以可以使用链式写法在调用了`then`方法后再调用`then`方法；另外`catch`方法是在**抛出异常**或**执行`reject`**时被调用的，其实和`then`方法传入错误处理函数是一样的。

**Promise 建立后会立即执行**，`then`指定的回调函数，将在**当前脚本所有同步任务执行完后**才会执行。

```javascript
new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
}).then(function() {
  console.log('resolved.');
});
console.log('Hi!');
// Promise
// Hi!
// resolved
```

**`resolve`函数的参数可以是另一个`Promise`的实例**。

```javascript
const p1 = new Promise(function(resolve, reject) {
  // ...
});
const p2 = new Promise(function(resolve, reject) {
  // ...
  resolve(p1);
});
```

这种情况`p1`的状态会传递给`p2`，导致了`p2`状态由`p1`决定，如果`p1`是`pending`，那么`p2`会等待`p1`的状态改变；如果`p1`状态是另外两种状态，那么`p2`的回调函数会立刻执行。

```javascript
const p1 = new Promise(function(resolve, reject) {
  setTimeout(() => reject(new Error('fail')), 3000);
});

const p2 = new Promise(function(resolve, reject) {
  setTimeout(() => {
    console.log('???');
    resolve(p1);
  }, 1000);
});

p2.then(result => console.log(result)).catch(error => console.log(error));
// ???
// Error: fail
```

上面代码中，`p1`是一个`Promise`，3 秒之后变为`rejected`。`p2`的状态在 1 秒之后改变，`resolve`方法返回的是`p1`。由于`p2`返回的是另一个 `Promise`，导致`p2`自己的状态无效了，由`p1`的状态决定`p2`的状态。所以，后面的`then`语句都变成针对后者（`p1`）。又过了 2 秒，`p1`变为`rejected`，导致触发`catch`方法指定的回调函数。

另外，调用`resolve || reject`不会终止`executor`函数的执行，但是一般情况下调用了这俩函数之后就应该结束执行了，所以一般在调用这两个函数之后就`return`。

#### Promise.prototype.then

前文提到了`then`方法可以用链式写法来调用，因为`then`方法会返回一个新的`Promise`对象；依照上面一段的代码，后一个`then`方法（或`catch`）会等待前一个的状态发生改变才会调用。

#### Promise.prototype.catch

`catch`方法可以说是`then(null, rejection)`的别名，用于指定发生错误时的回调函数。前面的例子也提到过了。但是要注意这个方法会捕获`then`方法抛出的异常，也更像是`try/catch`块的写法。因此建议都用下面代码的第二种写法，避免用`then(null, rejection)`方法。

```javascript
// bad
promise.then(
  function(data) {
    // success
  },
  function(err) {
    // error
  }
);

// good
promise
  .then(function(data) {
    //cb
    // success
  })
  .catch(function(err) {
    // error
  });
```

#### Promise.prototype.finally

这个方法用于指定无论`promise`实例的状态如何都会执行的方法。有点像默认方法的意思，且`finally`方法总是返回原来的值。

```javascript
// resolve 的值是 undefined
Promise.resolve(2).then(
  () => {},
  () => {}
);

// resolve 的值是 2
Promise.resolve(2).finally(() => {});

// reject 的值是 undefined
Promise.reject(3).then(
  () => {},
  () => {}
);

// reject 的值是 3
Promise.reject(3).finally(() => {});
```

上述代码描述了执行`finally`方法和`then`方法的区别，`then`方法会返回其方法本身返回的值，而`finally`方法会返回原来的值。

#### Promise.all Promise.race

两个方法的参数都是一组`Promise`实例，都是将这些实例包装成一个新的`Promise`实例。

方法的区别在于，`all()`方法会等所有参数的状态都变成`fulfilled`，新实例的状态才会变为`fulfilled`，或者有一个参数的状态变成`rejected`，实例的状态就变为`rejected`；`race()`方法的某个参数的状态改变，实例的状态都会改变。

`all`方法在`fulfilled`状态下会返回一个数组，数组元素是各个参数的返回值；在`rejected`状态下会返回第一个被`rejected`参数的返回值。`race`方法的返回值是最先改变状态的参数的返回值。

一个应用场景：

```javascript
const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function(resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000);
  })
]);

p.then(console.log).catch(console.error);
```

上述代码如果在 5 秒内`fetch`方法没得到返回，就会返回`rejected`，实例`p`的状态就会变成`rejected`

> 如果实例中有自己的`catch`方法，则不会将异常抛出到`all | race`方法中，且此时实例的状态都会变成`fulfilled`；

来看个例子：

```javascript
const p1 = new Promise((res, rej) => {
  rej('rejected');
}).catch(e => e);
const p2 = new Promise((res, rej) => {
  res('resolved');
});
Promise.all([p1, p2])
  .then(result => {
    console.log(result);
  })
  .catch(err => {
    console.log(err);
  });
// ["rejected", "resolved"]
```

可以看出如果在子`Promise`实例中已经`catch`了异常，在`all`方法里就不会再次捕获。

#### Promise.allSettled

新增的方法，该方法接受一组`Promise`实例参数，只有参数都返回结果后（不论是`fulfilled`还是`rejected`），才会返回一个新的`Promise`实例，并且返回实例的状态总是`fulfilled`；由于返回值是实例对象，所以还需要访问`then`方法去取得返回数组，来看例子：

```javascript
const p1 = new Promise((res, rej) => {
  rej('rejected');
}).catch(e => e);
const p2 = new Promise((res, rej) => {
  res('resolved');
});
Promise.allSettled([p1, p2]).then(result => {
  console.log(result);
});

let num = Math.random();
const p3 = new Promise((res, rej) => {
  if (num >= 0.5) {
    res(num);
  } else {
    rej(num);
  }
});
num = Math.random();
const p4 = new Promise((res, rej) => {
  if (num >= 0.5) {
    res(num);
  } else {
    rej(num);
  }
});
let settled = await Promise.allSettled([p3, p4]);
console.log(settled);
// 0: {status: "rejected", reason: 0.3948534817154049}
// 1: {status: "fulfilled", value: 0.903275305994583}

// 0: {status: "fulfilled", value: "rejected"}
// 1: {status: "fulfilled", value: "resolved"}
```

可以看到返回的`Promise`实例中其实包含了返回值数组，数组元素都是对象，对应了执行参数后的返回值，`status`属性就是当前实例的状态，值是字符串`fulfilled | rejected`的其中一个；当状态是`fulfilled`时，对象有`value`属性，可以看到值是`res`函数传入的参数；状态是`rejected`时，对象有`reason`属性，值是`rej`传入的参数。

另外第一段代码，`p1`调用了`rej`方法然后`catch`了，这里就会返回新的`Promise`实例且实例的状态为`fulfilled`，所以调用`allSettled`方法后返回的对象状态属性还是`fulfilled`；

#### Promise.any

新增的方法，接受一组`Promise`实例作为参数，只要有一个变成`fulfilled`状态，返回的实例就是`fulfilled`状态；如果所有的实例都变成`rejected`状态，那么返回的实例就是`fulfilled`状态；这个方法还处于提案阶段所以无法检验。

> 这几个方法类似于逻辑电路里的与非门，`all`方法是与的关系，并且有短路运算的特性；`any`方法是或的关系，不具备；`race`方法只是看哪个方法最快返回而不管返回状态；`allSettled`方法会等待所有参数状态确定

#### Promise.resolve

该方法用于将对象转化成`Promise`对象；根据参数的不同，总共有四种情况：

- Promise 实例

由于参数本身就是实例对象了，所以会直接返回这个实例；

- `thenable`对象

`thenable`对象是具有`then`方法的对象，`resolve`方法会把这个对象转换成`Promise`对象然后立即执行`then`方法，看个例子：

```javascript
const thenable = {
  num: Math.random(),
  then(res, rej) {
    if (this.num >= 0.5) {
      res(this.num);
    } else {
      rej(this.num);
    }
  }
};
let p = Promise.resolve(thenable);
p.then(result => {
  // return result;
  console.log('resolved', result);
}).catch(err => {
  console.log('rejected', err);
  return err;
});
```

- 参数没有`then`方法

这种情况包括了没有`then`方法的对象，原始值（数字、字符串等），此时调用`resolve`方法会返回一个新的`Promise`对象，状态为`resolved`，传入的参数作为`value`，看个例子：

```javascript
let p = Promise.resolve('?');
p.then(val => {
  console.log(val);
});
// 等价于
let p = new Promise((res, rej) => {
  res('?');
});
p.then(val => {
  console.log(val);
});
```

- 没有传入参数

不传入参数时，该方法直接返回一个`resolved`状态的`Promise`对象，比起`new`一个新的实例，再去调用`res`方法，这个方法比较简单；

```javascript
setTimeout(function() {
  console.log('three');
}, 0);

Promise.resolve().then(function() {
  console.log('two');
});
console.log('one');
// one
// two
// three
```

可以看到立即执行`then`方法是微任务，会在宏任务执行完一轮后集中执行，且是在事件循环结束时执行，`setTimeout`方法会在下一轮时间循环开始时执行，所以会先输出`two`；

#### Promise.reject

该方法一样会返回一个新的`Promise`实例，实例状态为`rejected`；函数参数是`reason`，即含义为失败的原因，不区分参数类型，不对参数进行处理，看个例子：

```javascript
const o = {
  then(res, rej) {
    res();
  }
};
const s = 'rejected reason';
let p1 = Promise.reject(o);
let p2 = Promise.reject(s);
p1.catch(e => {
  console.log(Object.is(o, e));
  // true
  return Object.is(o, e);
});
p2.catch(e => {
  console.log(e);
  // rejected reason
  return e;
});
```

可以看出上面的代码中，即使参数是对象，也会原封不动的传递给`catch`函数

### 应用场景

#### 加载图片

```javascript
const preloadImage = function(path) {
  return new Promise(function(resolve, reject) {
    const image = new Image();
    image.onload = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
preloadImage('http://pic75.nipic.com/file/20150821/9448607_145742365000_2.jpg')
  .then(() => {
    console.log('?');
  })
  .catch(() => {
    console.log('??');
  });
```

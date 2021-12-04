## async 函数

async 函数是 ES2017 标准引入的，看关键字就可以想到是用于异步操作的函数，我们在上一章讲过了 Generator 函数的异步用法，为什么还要引入新的函数呢，事实上 async 函数并不是新的，只是 Generator 函数的语法糖，并且在其基础上做了一些改进。

### 含义

上一章开头我们写了一个 ajax 的例子，这章接着用这个例子来讲解；

```javascript
const getData = (method, url) => {
  return new Promise((res, rej) => {
    let xhr = new XMLHttpRequest();
    xhr.open(method, url);
    xhr.send();
    xhr.onreadystatechange = () => {
      if (xhr.readyState == 4 && xhr.status == 200) {
        // console.log(xhr.responseText);
        res(xhr.responseText);
      }
    };
  });
  xhr.addEventListener('error', err => {
    console.log(err);
    rej(err);
  });
};
let gen = function*() {
  let r1 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r1);
  let r2 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r2);
};
let g = gen();
g.next().value.then(res => {
  console.log(res);
});
```

用 async 函数的写法：

```javascript
let gen = async function() {
  let r1 = await getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r1);
  let r2 = await getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r2);
};
let g = gen();
```

对比发现把`*`表达式替换成了`async`关键字，`yield`关键字替换成了`await`；

#### 一些改进

1. 内置执行器

上一章提到过，Generator 函数执行需要手动调用`next`方法，或者用`co`模块，但是 async 函数自带执行器，它的执行和普通函数一样，直接调用即可`gen()`；

2. 语义化

对于异步操作来说，比起 Generator 函数的奇怪语法，`async`和`await`明显更加语义化，一看就知道是用来做异步操作和等待执行的；

3. 适用性

上一章讲自动执行器时，`co`模块的 Generator 函数，`yield`关键字后面的表达式只能是`thunk | promise`函数或者对象，但 async 函数的`await`关键字后面可以接`Promise | primitive`对象，原始类型会调用`Promise.resolve`方法自动转换成`Promise`对象。

4. 返回 Promise

async 函数返回值是 Promise 对象，Generator 函数返回值是 Iterator 对象，各有千秋；async 函数可以看做内部多个异步操作，包装成一个 Promise 对象，`await`关键字是其内部的`then`方法的语法糖。

### 基本用法

async 函数返回一个`Promise`对象，可以用`then`方法添加回调函数，注意是函数调用后返回的`Promise`对象；当函数执行时，一旦遇到`await`就会先返回`Promise`的实例，此时实例状态为`pending`，因为还在等待`await`后面的异步操作执行完毕，我们在前面也提到过`await`后面跟的表达式会被转换成`Promise`对象，所以这里其实相当于返回的`Promise`实例中嵌套了`await`表达式的`Promise`对象，在`Promise`那一章我们说过，外层会等待内层的状态改变为`resolved | rejected`，才会接着执行；所以等内部的异步操作执行完了，才会再执行函数体内后面的语句，看个例子：

```javascript
async function timeout(ms) {
  await new Promise(resolve => {
    setTimeout(() => {
      resolve(1);
    }, ms);
  }).then(res => {
    console.log(res);
  });
  return 10;
}

async function asyncPrint(value, ms) {
  let t = await timeout(ms);
  console.log(t);
  console.log(value);
}
asyncPrint('hello world', 500);
// 1
// 10
// hello world
```

首先这段代码有两个`async`函数，先来看`timeout`；`await`跟了一个`Promise`实例，作用是延迟一段时间执行传入的`resolve`函数，`resolve`函数传了个参数`1`，在随后的`then`方法中把参数打印出来了，函数最后一步返回了`10`；`asyncPrint`调用了`timeout`函数，注意这个函数调用是跟在`await`后面的，所以运行到这的时候会先返回`Promise`实例，状态是`pending`，因为取决于`timeout`函数的状态变化；执行`timeout`函数，延迟`500ms`后打印出参数`1`，返回值`10`，被赋值给`t`，打印出`10`，最后打印出`hello world`。

分析后不难发现，`await`关键字会使当前函数等待内部的异步函数执行完毕，才接着执行；

另一个要提出来的点就是，`await`整个表达式的值，取决于`await`表达式执行后的返回值，比如上面的代码明确了返回值是`10`；如果没有明确的`return`语句，那么得到的就是`undefined`了，和函数没有返回值是一样的，合理；

如果`asyncPrint`函数中`timeout`调用时没有`await`关键字，那么`t`应该是一个`Promise`对象实例，看例子：

```javascript
async function timeout(ms) {
  await new Promise(resolve => {
    setTimeout(() => {
      resolve(1);
    }, ms);
  }).then(res => {
    console.log(res);
  });
  return 10;
}

async function asyncPrint(value, ms) {
  let t = timeout(ms);
  console.log(t);
  console.log(value);
}
asyncPrint('hello world', 500);
// Promise {<pending>}
// hello world
// 1
```

由于不会等待`timeout`执行，所以会直接返回`Promise`实例并赋值给`t`，此时状态为`pending，因为`timeout`内部有`await，和我们上面说的遇到`await`返回`Promise`实例是一致；外层函数`asyncPrint`正常执行到结束，内部函数`timeout`延迟了`500ms`打印出了`1`，返回值`10`就没有用了，因为没有什么地方可以接收这个返回值了。

#### async 函数的使用形式

上面用的基本都是普通声明函数式的方式，看接下来的代码：

```javascript
async function name(params) {}
const name = async function(params) {};
let obj = {
  async name() {}
};
class P {
  constructor() {}
  async name() {}
}
const name = async () => {};
```

涵盖了大部分的使用场景的词法。

### 重点语法

#### 返回 Promise 对象

上面说过了哈，反正就是会返回一个`Promise`对象，并且内部的`return`语句返回的那个值，会被当做`then`方法回调函数的参数；上面的例子也表明了，如果 async 函数调用时前面还有`await`参数，那会等到内部函数执行完了之后再看返回值；看例子：

```javascript
async function timeout(ms) {
  await new Promise(resolve => {
    setTimeout(() => {
      resolve(1);
    }, ms);
  }).then(res => console.log(res));
  return 10;
}

async function asyncPrint(value, ms) {
  let t = timeout(ms);
  // console.log(t);
  t.then(res => console.log(res));
  console.log(value);
  return 20;
}
asyncPrint('hello world', 500).then(res => console.log(res));
// hello world
// 20
// 1
// 10
```

俩 async 函数都用了`return`语句，都作为了`then`方法回调函数的参数；

#### 返回对象的状态变化

前面也提过了哈，async 函数返回的 Promise 对象，是跟内部的异步函数状态有关的，像嵌套的`Promise`对象一样，要等所有内部`await`命令后的 Promise 对象执行完，才会发生状态改变，或者遇到`return`语句抛出错误语句，例子上面也有了；

#### await 表达式

`await`表达式后面一般是一个 Promise 对象，返回该对象的结果；如果不是，则会转换成 Promise 对象，基本类型的值转换成 Promise 对象后，会直接返回对应的值本身；在 Promise 那篇里提过如果对象属性里有`then`方法，也可以直接转换成 Promise 对象，这里也一样；

```javascript
(async () => {
  return await 123;
})().then(console.log);
// 123
```

再来看个使用 async 函数实现`Sleep`函数的方法；

```javascript
class Sleep {
  constructor(timeout) {
    this.timeout = timeout;
  }
  then(res, rej) {
    setTimeout(res, this.timeout);
  }
}
(async function sleepTest() {
  for (let index = 0; index < 5; index++) {
    console.log(index);
    await new Sleep(200);
  }
})();
```

另外`await`关键字后面的 Promise 对象如果是`reject`状态，则`reject`参数会被`catch`方法的回调函数接收；

```javascript
async function f() {
  await Promise.reject('?');
  await Promise.resolve('?');
}
f()
  .then(r => console.log('resolve', r))
  .catch(e => console.log('error', e));
// error ?
```

可以看到在`reject`语句前没有`return`语句，但是在`catch`方法的回调函数里也接收到了参数；在`reject`语句执行了之后，整个`async`函数都会中断执行，不会再往下。

#### 错误处理

像上面提到的这种情况，有时候我们希望一条语句的`reject`不会影响后面的`await`表达式的执行，所以此时我们会把需要忽略错误的的`await`语句放到`try catch`语句里；

```javascript
async function f() {
  try {
    await Promise.reject('?');
  } catch (e) {
    console.log(e);
  }
  return await 123;
}
f().then(console.log);
// ?
// 123
```

Promise 对象内部用`throw`方法抛出的异常也一样，用`try catch`块去捕获，就不会中断当前函数执行了；总结起来就是内部抛错内部处理。

多个互相影响的`await`语句可以放在一个`try catch`块里，而不想被影响的`await`语句就放在外面，分个类，好操作；

有时候可能会遇到需要轮询的场景，报错就继续下一轮，如果得到了返回值就跳出轮询，也可以用这种方式了，看个例子：

```javascript
async function polling(counts) {
  for (let i = 0; i < counts; i++) {
    try {
      await fetch('https://www.baidu.com');
      break;
    } catch (error) {
      console.log(error);
    }
  }
  console.log('?');
}
polling(3);
```

尝试了 3 次`fetch`一个链接，如果有返回值就`break`循环，错误了就会开启下一轮循环。

#### 一些注意点

- 不相关的异步操作不要写成有先后顺序的关系；

其实 async 函数本身就是为了解决异步操作的同步性问题，就是俩异步操作存在先后关系时，我们才会用到 async 函数，没有这种先后关系，自然是直接用异步咯，这样在同一个时间段内可以执行多段任务提高效率；写法：

```javascript
// 不建议
await unrelated();
await unrelated2();

// 建议
await Promise.all([unrelated(), unrelated2()]);

// 或者
let x = unrelated(),
  y = unrelated2();
let r1 = await x;
let r2 = await y;
```

说白了就是让异步过程保持它们原本的工作顺序。

- `await`命令只能在`async`函数中用，不能在普通函数中用

- async 函数可以保留运行时的堆栈

我们很多时候会在一个同步函数里执行一个异步函数，想得到它的返回值，但很多时候异步函数还没执行完，同步函数却先执行完了，这就很僵了(说了很多次 async 函数就是为了解决这个问题。。)；那其实直接用 async 函数就行了，反正都会卡在`await`语句上，等返回结果了才会接着执行，所以如果在函数内部报错了，那么错误堆栈将会包括外层函数，因为此时外层函数的上下文还没有消失。

### async 函数的实现原理

其实一开头就说过了，async 函数是 Generator 函数的语法糖，Generator 函数+co 自动执行器就是 async 函数了；

```javascript
/**
 * @param {function} gen generator function
 * @return {promise} promise instance
 * @author: youzi
 * @Date: 2020-01-20 10:01:24
 */
function spawn(gen) {
  return new Promise(function(resolve, reject) {
    // g 是Generator函数的遍历器对象
    const g = gen();
    // 判断g是不是正常的返回值
    if (!g || typeof g.next !== 'function') {
      return res(g);
    }
    // 递归主体
    /**
     * @param {function} nextF 执行并返回g.next()
     * @return {promise}: promise对象
     * @author: youzi
     * @Date: 2020-01-20 11:13:04
     */
    function step(nextF) {
      // next用来保存每次调用g.next()的结果
      let next;
      try {
        next = nextF();
      } catch (e) {
        // next方法出错时的回调
        return reject(e);
      }
      // 判断递归是否正常结束
      if (next.done) {
        return resolve(next.value);
      }
      // 手动调用Promise.resolve方法构造promise对象，对象状态为resolved
      Promise.resolve(next.value).then(
        // 状态为resolved时的回调
        // 参数v就是next.value
        function(v) {
          // 递归调用step函数，传入函数
          step(function() {
            return g.next(v);
          });
        },
        function(e) {
          // 异常捕获函数
          step(function() {
            return g.throw(e);
          });
        }
      );
    }
    // 第一次手动进行调用，传入参数为空
    step(function() {
      return g.next(undefined);
    });
  });
}
```

其实和上一篇写过的 co 执行器的代码差不多吧。

### 对比其他异步方法

目前 ES6 以后部署的异步方法主要是 Promise，Generator 函数，async 函数，部署回调函数。

写几个用这些函数的通用模式，来对比一下：

```javascript
// Promise
function name(asyncMethods) {
  let result = null;
  let p = Promise.resolve();
  for (const method of asyncMethods) {
    p = p.then(val => {
      result = val;
      return method();
    });
  }
  return p
    .catch(e => {
      // do something
    })
    .then(() => result);
}

// Generator
function name(asyncMethods) {
  return spawn(function*() {
    let result = null;
    try {
      for (const method of asyncMethods) {
        result = yield method();
      }
    } catch (error) {}
    return result;
  });
}

// async
async function name(asyncMethods) {
  let result = null;
  try {
    for (const method of asyncMethods) {
      result = await method();
    }
  } catch (error) {}
  return result;
}
```

反正对比起来最简洁的还是 async 函数，Generator 函数用来写异步操作时往往需要自己提供自动执行器，Promise 对象自己写的代码就更多了。

### 总结

总的来说，涉及到异步操作同步化的情况，都最好用 async 函数咯，方便好用，代码语义化强，易读性强。

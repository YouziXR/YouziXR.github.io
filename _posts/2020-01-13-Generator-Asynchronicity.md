## Generator 函数在异步编程中的应用

众所周知 JS 是单线程语言，同一时刻只有一个任务在执行，所以就出现了异步编程的方式，在 ES6 之前已经有了异步编程的解决方案，而 Generator 函数也提供了异步操作的方法；

### 异步

异步(Asynchronicity)指的是任务的不连续性，一个异步任务通常可以被分成多段执行，期间可以穿插其他任务作为前置或者后置；

举个例子，比如 A 问 B 什么是异步编程，A 说我要查查百度，A 去查百度了，B 这时候不会一直干等着 A 的结果，而是转头去做其他事，等 A 有结果了，B 再回头接受 A 的结果；我们的程序相当于 B，如果在等结果时程序一直卡着，那也太蠢了。当然这只是我个人理解，异步这个概念也不能就这样简单的解释，还得在自己写代码的时候体会体会。

### ES6 前的异步操作方式

- 回调函数方式
- 事件监听方式
- 发布/订阅模式
- Promise 对象

#### 回调函数

JS 实现异步编程就是靠回调函数(`callback`)，异步函数不是可以分为多段执行吗，回调函数就是第二段第三段...操作过程，像是 ajax 请求，是一个异步的过程，简单地把请求进行一次封装；

```javascript
function loadData(method, url, callback) {
  let xhr = new XMLHttpRequest();
  xhr.open(method, url);
  xhr.onreadystatechange = callback;
  xhr.send();
}
function callback() {
  if (this.readyState == 4 && this.status == 200) {
    console.log(this.responseText);
  }
}
loadData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js', callback);
```

可以看到我们传入的第三个参数就是一个回调函数，指明了当请求成功之后应该接着执行哪些操作；明显整个任务分成了几段，第一段先调用了`open`方法，然后调用了`send`方法发送请求，发送过程是一个异步的操作，因为程序执行不会卡着一直等待请求的返回结果，而是会继续执行其他的任务；这时回调函数作用就体现了，`onreadystatechange`是一个在`readyState`属性改变时会触发的函数，类似监听器，我们提前定义好了这个函数要执行什么操作，在第一段任务做完了之后，就会执行第二段任务，期间有其他任务接着执行，不会浪费程序资源；

很多情况下我们会给回调函数传入一些参数，比如错误码之类的信息，这是因为任务的上下文（context）已经变了，在回调函数内部访问不到外部的错误信息，所以需要用参数的方式传入；

#### promise

多数情况下回调函数嵌套层数不会过深，但如果有这种情况，那么代码写起来就有点痛苦了；你可能会在回调里面还有回调，比如在 ajax 请求完成之后还有其他的 ajax 请求，这些回调函数导致异步操作耦合度变高，代码维护就比较困难了，相信大家都听过回调地狱(callback hell)的说法，指的就是这种情况了。

Promise 对象就是为了解决这种问题的，其实不是一种新的语法，而是一种新的写法，允许回调函数嵌套，改成链式调用，这样代码就会比较清晰，看个多重回调的例子：

```javascript
const load = (method, url) => {
  /* xhr.onreadystatechange = () => {
  }; */
  let p = new Promise((res, rej) => {
    let xhr = new XMLHttpRequest();
    xhr.open(method, url);
    xhr.onreadystatechange = () => {
      if (xhr.readyState === 4 && xhr.status == 200) {
        res(xhr.responseText);
      }
    };
    xhr.send();
  });
  return p;
};
load('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js')
  .then(res => {
    console.log(res);
  })
  .then(() => {
    return load('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  })
  .then(res => {
    console.log(res);
  });
```

上面代码写了好几个回调函数，都以`then`方法的形式呈现，错误可以用`catch`方法去捕获；其实这样的写法只是让代码起来清晰了，本质没有变化，各个任务的子任务都单独写在每个`then`方法里了，而其实这样的写法也是代码段里都是`then`，看着也头疼，下面来看 Generator 函数是如何解决这类问题的。

#### Generator 函数

Generator 函数其实是借鉴了一种异步编程方案，协程(coroutine)，指多个线程相互协作完成异步任务；其运行流程大致是：

- 协程 ① 开始执行；
- 协程 ① 执行到一半，暂停，执行权转交给协程 ②；
- 协程 ② 执行一段时间后交还执行权；
- 协程 ① 继续执行；

其实看着上面的描述，是不是很像 Generator 函数的执行过程，调用`next`方法，开始执行协程 ①，遇到`yield`表达式，执行完表达式语句后就暂停执行，转而执行协程 ②，直到再一次调用`next`方法，就恢复协程 ① 的执行；其次这种写法也很像是同步函数，除了前面多了个`yield`关键字，很清晰的可以看出执行顺序，代码维护性就提高了。就不举例了，上一篇写了很多。

另外由于 Generator 函数也有差错控制，所以就不必像回调函数那样把`Error`对象作为参数传入了，直接在 Generator 函数内部使用`try catch`代码块就能捕获内部报错信息了。

### thunk 函数

thunk 函数是自动执行 Generator 函数的一种方式；thunk 函数是一个临时函数，用于调用返回普通函数的参数表达式结果；看个例子：

```javascript
const f = m => m + 2;
let x = 1;
f(x + 1);
// 在JS里这段代码执行时相当于
const thunk = () => x + 1;
const f = thunk => thunk() + 2;
```

JS 用一个临时函数保存了普通函数的参数表达式，并在调用时将这个表达式的结果运算后返回；所以 thunk 函数是用来替换一个表达式的。

那在 JS 里，thunk 函数被用于替换一个，含有回调函数参数的多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数，看我们最开始写的那个例子：

```javascript
function loadData(callback) {
  return (method, url) => {
    let xhr = new XMLHttpRequest();
    xhr.open(method, url);
    xhr.onreadystatechange = callback;
    xhr.send();
  };
}
function callback() {
  if (this.readyState == 4 && this.status == 200) {
    console.log(this.responseText);
    return this.responseText;
  }
}
loadData(callback)('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
```

一个通用的 thunk 函数转换器，只要是带有回调函数参数的函数都可以用这个方法转换成 thunk 函数；

```javascript
const Thunk = function(fn) {
  return function(...args) {
    return function(callback) {
      // return fn.call(this, ...args, callback);
      return Reflect.apply(fn, this, [...args, callback]);
    };
  };
};
Thunk(loadData)('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js')(callback);
```

在生产环境下可以直接选择用`Thunkify`模块，使用`npm`安装即可；

乍一看是不是感觉没什么用，做这样的转换意义不大，但是在 Generator 函数出现后就有意义了，我们可以利用 thunk 函数对异步函数自动执行；

#### Generator 函数的流程管理

我们知道如果 Generator 函数中的所有操作都是同步的，那么直接用循环方法就能自动执行这些操作了，但是如果有异步的操作，就必需手动调用，使用 thunk 函数可以实现异步函数的自动执行；

先看个手动执行的例子，还是用最开始的 ajax 请求的函数；

```javascript
function loadData(method, url, callback) {
  let xhr = new XMLHttpRequest();
  xhr.open(method, url);
  xhr.onreadystatechange = callback;
  xhr.send();
}
const Thunk = function(fn) {
  return function(...args) {
    return function(callback) {
      // return fn.call(this, ...args, callback);
      return Reflect.apply(fn, this, [...args, callback]);
    };
  };
};
const getData = Thunk(loadData);
let gen = function*() {
  let r1 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r1);
  let r2 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r2);
};
let g = gen();
let r1 = g.next();
r1.value(function() {
  if (this.readyState == 4 && this.status == 200) {
    // console.log(this.responseText);
    let r2 = g.next(this.responseText);
    r2.value(function() {
      if (this.readyState == 4 && this.status == 200) {
        g.next(this.responseText);
      }
    });
  }
});
```

可以看到`loadData`函数有回调函数作为参数，所以可以调用`Thunk`函数让它变成一个 thunk 函数；处理好了 thunk 函数后，构造 Generator 函数，第一个异步操作是调用`getData`函数，这个函数调用后返回一个接受回调函数作为参数的函数，所以在外层的`r1`变量的`value`属性是个函数，我们在外层调用了`value`，传入回调函数，此时才会执行`Thunk`函数的最内层的`Reflect.apply`方法，也可以说这时候才会真正的执行`loadData`函数；

在回到函数体中，第二次调用了`next`方法，并且传入参数(这里的参数是 ajax 请求后的返回值)，此时`gen`函数内部的`r1`就等于传入的参数值，遍历器执行到第二个`yield`语句，和前面一样的操作。

明显地，我们将一个回调函数递归传入到`next`方法的`value`属性，这可以用递归函数做到；

#### Generator 函数流程自动化

重新构造一个递归函数，参照上面的手动调用代码，要求每个`yield`关键字后面的表达式必须是 thunk 函数；

```javascript
/**
 * @param {function} gen generator function
 * @author: youzi
 * @Date: 2020-01-15 17:13:15
 */
const run = gen => {
  let g = gen();
  let result = g.next();
  function callback() {
    if (this.readyState == 4 && this.status == 200) {
      result = g.next(this.responseText);
      if (result.done) {
        return;
      }
      result.value(callback);
    }
  }
  result.value(callback);
};
function loadData(method, url, callback) {
  let xhr = new XMLHttpRequest();
  xhr.open(method, url);
  xhr.onreadystatechange = callback;
  xhr.send();
}
const Thunk = function(fn) {
  return function(...args) {
    return function(callback) {
      return Reflect.apply(fn, this, [...args, callback]);
    };
  };
};
const getData = Thunk(loadData);
let gen = function*() {
  let r1 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r1);
  let r2 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r2);
};
run(gen);
```

用于递归的函数是`run`，传入一个 Generator 函数，调用这个函数取得遍历器对象，手动调用第一次`next`方法，因为第一次调用`next`方法不需要传入参数；定义递归函数`callback`，这个函数同时也是回调函数，递归函数的主要逻辑就是调用`next`方法，并传入需要的参数，判断 Generator 函数是否已经执行完毕(就是判断返回对象的`done`属性)，如果没有执行完毕就接着调用返回对象的`value`属性，上面已经讲过，`value`属性就是接受回调函数的函数，所以再将`callback`作为参数传入，实现递归；`run`函数的最后一步是因为我们手动调用了第一次的`next`方法，所以这里也得手动调用返回对象的`value`方法。

### co 模块

co 模块是一个用于 Generator 函数自动执行的工具，可以不用再手动写`run`函数了，和`run`函数一样的用法，传入 Generator 函数作为参数，返回一个`promise`对象，可以用`then`方法来添加执行成功的回调函数；

co 模块的原理其实是构造了一个自动执行器，上面提到的 thunk 函数是一种模式，另一种模式是采用`Promise`对象，co 模块将这两种模式包装成一个模块，实现自动执行异步操作，下面来介绍基于`Promise`对象的自动执行器。

### 基于 Promise 对象自动执行器

还是用 ajax 的例子，我们把请求函数包装成一个`Promise`对象看看；

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
};
let gen = function*() {
  let r1 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r1);
  let r2 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r2);
};
let g = gen();
g.next().value.then(res => {
  console.log('?');
  g.next(res).value.then(res => {
    console.log('??');
    g.next(res);
  });
});
```

一个很简单的手动调用 Generator 函数的例子，其中返回对象的`value`属性变成了`Promise`对象，所以在`value`属性后面加上`then`方法来定义请求成功的回调函数；不难看出这和 thunk 函数差不多也可以用递归形式来改写为自动执行；

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
};
let gen = function*() {
  let r1 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r1);
  let r2 = yield getData('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  console.log(r2);
};
const run = gen => {
  let g = gen();
  let result = g.next();
  function thenMethod(res) {
    console.log('?');
    result = g.next(res);
    if (result.done) {
      return result.value;
    }
    result.value.then(res => {
      thenMethod(res);
    });
  }
  result.value.then(res => {
    thenMethod(res);
  });
};
run(gen);
```

和 thunk 函数的模式差不多，只是要求`yield`关键字后的表达式返回`Promise`对象。

#### co 模块源码分析

co 模块是上面两个自动执行器的扩展，源码很简短；

```javascript
function co(gen) {
  let that = this;
  return new Promise((res, rej) => {
    let g;
    if (typeof gen === 'function') {
      g = gen.call(that);
    }
    if (!g || typeof g.next !== 'function') {
      return res(g);
    }

    (function onFulfilled(res) {
      let ret;
      try {
        ret = g.next(res);
      } catch (error) {
        return rej(error);
      }
      thenMethod(ret);
    })();
    function thenMethod(ret) {
      if (ret.done) {
        return res(ret.value);
      }
      let value = Promise.resolve(ret.value);
      if (value) {
        return value.then(onFulfilled, onRejected);
      }
      return onRejected(new Error('?'));
    }
  });
}
```

首先要判断传入参数是否是 Generator 函数，是的话执行这个函数，得到遍历器对象，否则返回`Promise`对象，且状态为`resolved`；

`onFullfilled`函数用于封装`thenMethod`，控制`gen`函数往下执行，并且在函数内部使用`try catch`块来捕获内部的错误；

`thenMethod`函数是递归函数，先检查是不是`gen`函数的最后一步；如果不是，再让`value`属性转换成`promise`对象，调用`then`方法，传入的`resolve`函数就是`onFullfilled`，这样就实现了递归调用`thenMethod`函数，如果`value`属性无法转换成`promise`对象，那么就返回错误函数；这里主要是在参数不符合要求时会抛出错误，如果不是 thunk 函数和`promise`对象。

### 并发的异步操作

co 操作支持并发的异步操作，允许某些操作同时进行，等到并发全部完成，才会进入下一步；具体语法是把并发操作放在数组或对象里，跟在`yield`语句后面。

```javascript
// 数组的写法
co(function*() {
  let res = yield [Promise.resolve(1), Promise.resolve(2)];
  console.log(res);
}).catch(onerror);
// 对象的写法
co(function*() {
  let res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2)
  };
  console.log(res);
}).catch(onerror);
```

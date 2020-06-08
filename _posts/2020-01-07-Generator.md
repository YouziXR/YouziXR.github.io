# Generator

Generator 是一个函数的类型，是 ES6 提供的一种异步编程方案；之所以说是异步的，原因是其函数内部封装了多种状态，函数并不会顺序执行，而是需要手动调用函数的`next`方法，从上一个状态，转移到下一个状态，期间会执行状态转移的中间代码，是不是和`Iterator`接口很像；

和`Iterator`接口更像的地方是，Generator 函数会返回一个遍历器对象，可以调用这个对象的`next`方法依次访问函数内部的每个状态，所以其实也可以用遍历方法比如`for of`去访问 Generator 函数的每个状态；

## 语法

在定义 Generator 函数时，在`function`关键字和函数名之间添加了一个`\*`符号来区别普通函数和 Generator 函数；然后在函数内部，使用`yield`表达式来定义不同的函数状态，yield 是产出的意思，Generator 是生成器的意思；

```javascript
const func1 = () => {
  return '1';
};
const func2 = () => {
  return '2';
};
function* gen() {
  console.log('?');
  yield func1();
  console.log('??');
  yield func2();
  console.log('???');
  return '100000';
}
let i = gen();
i.next();
// '?'
// {value: "1", done: false}
i.next();
// '??'
// {value: "2", done: false}
i.next();
// '???'
// {value: "100000", done: true}
i.next();
// {value: undefined, done: true}
i.next();
// {value: undefined, done: true}
```

上面的代码声明了三个函数，两个普通函数`func1 | func2`，一个 Generator 函数`gen`；Generator 函数有三种状态，两个`yield`语句的状态，一个`return`语句的状态，我们暂时先称这三个状态为状态 1、2、3；

执行`gen`函数时，会返回遍历器对象，所以第一条打印语句`?`还没有执行；此时调用`next`方法，函数会执行到第一个状态结束，也就是打印`?`，然后返回对象`{value: "1", done: false}`；一直调用`next`方法，执行到最后一个状态`return`时，此时已经遍历完了，所以会返回最后那个对象，以后再继续调用`next`方法，函数的状态都停留在结束的状态了；

### yield 表达式

其实这样执行完了一遍之后，基本对遍历器对象的`next`方法，和`yield`表达式的执行顺序和规则有点认识了，可以归纳总结为如下：

1. 第一次调用`next`方法时，执行函数到第一个状态结束；
2. 遇到`yield`表达式，执行紧跟在`yield`语句后的表达式，并作为返回对象的`value`属性值，同时暂停执行后续语句，也就是只执行到`yield`表达式这一句；
3. 下一次调用`next`方法时，就继续往下执行，直到遇到下一个`yield`表达式，其实就是执行到下一个状态结束；
4. 如果没有遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，将`return`语句后面的表达式的值，作为返回对象的`value`属性的值；
5. 如果函数没有`return`语句，那么返回的对象的`value`属性值就是`undefined`。

> `yield`紧跟的表达式，是一种惰性求值，只有当调用了`next`方法，遍历器对象内部指针指向这条语句时才会执行；

```javascript
function* gen() {
  yield 1 + 2 + 3;
}
```

> **yield 表达式本身总是返回`undefined`，或者说根本没有返回值**，下面这段代码表现了这个特性；

```javascript
function* gen() {
  let y = yield 'a';
  console.log(y);
}
let g = gen();
g.next();
// {value: "a", done: false}
g.next();
// undefined
// {value: undefined, done: true}
```

第一次打印出的`undefined`就是表达式返回给变量`y`的值。

#### 与 return 语句的对比

- 相似点：都返回紧跟在语句后面的表达式的值；
- 不同点：
  - 每次函数执行遇到`yield`关键字，会暂停执行，在下一次函数执行时从这个位置继续往下执行，可以说具有状态记忆功能，而`return`很明显没有；
  - 函数的`return`语句可以有多条，但是只会执行一条，毕竟执行`return`语句后函数执行就结束了，`yield`语句可以执行多条；
  - 一般函数只会有一个返回值（可能是各种数据类型），而 Generator 函数可以返回好几个值，也是因为可以有多个`yield`语句；

#### 非 Generator 函数不能使用 yield 表达式

虽然说在 Generator 函数中也可以不用`yield`表达式，此时函数只是一个单纯的暂缓执行函数，还是得调用`next`方法才会执行；但如果在普通函数中使用了`yield`表达式会抛出语法错误的异常；

#### 和其他表达式连用

如果`yield`表达式放在了其他表示中，需要加圆括号才能使用；

```javascript
function* gen() {
  console.log('1' + (yield 10));
}
let g = gen();
g.next();
// {value: 10, done: false}
g.next();
// 1undefined
// {value: undefine, done: true}
```

好像一般也没有人这样写。。

#### 与 Iterator 接口的关系

前面也提过，Generator 函数会生成一个遍历器对象，所以可以直接把 Generator 函数当成遍历器生成函数，也就是说可以给对象的`Symbol.iterator`属性赋值一个 Generator 函数，使得对象具有遍历器接口；

```javascript
let o = {
  a: 1,
  b: 2
};
o[Symbol.iterator] = function*() {
  // let keys = Reflect.ownKeys(this);
  let keys = Object.keys(this);
  for (let i = 0; i < keys.length; i++) {
    const element = this[keys[i]];
    yield element;
  }
};
for (const iterator of o) {
  console.log(iterator);
}
// 1
// 2
```

一个有意思的现象，Generator 函数执行后返回的遍历器对象，其本身也具有`Symbol.iterator`属性，执行后返回自身；

```javascript
function* gen() {}
let g = gen();
g[Symbol.iterator]() === g;
// true
```

### next 方法的参数

`next`方法可以传入参数，作为上一个`yield`表达式的返回值，这样我们可以利用这个参数达到不同时间向函数传入不同参数的目的；

```javascript
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}
let genObj = dataConsumer();
genObj.next();
// Started
genObj.next('a');
// 1. a
genObj.next('b');
// 2. b
```

可以看到第一次调用`next`方法是不传参数的，因为参数是作为上一个表达式的返回值；

### Generator 函数在遍历时的应用

由于 Generator 函数返回遍历器对象的特性，我们可以用`for of`循环来自动的遍历返回的遍历器对象，并且不需要手动调用`next`方法了；

```javascript
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}
for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

值得注意的是`for of`方法不会返回属性`done`的值为`true`的对象，所以上述代码不会把`return`语句的`6`打印出来；

在 Iterator 那篇博客里，我们写了在`Object`上部署`Symbol.iterator`接口，从而实现遍历对象结构；同样的我们也可以用 Generator 函数来添加这个接口，就可以用`for of`来循环遍历了；

```javascript
function* gen(obj) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const element = obj[key];
      yield element;
    }
  }
}
let o = {
  a: 1,
  b: 2
};
for (const i of gen(o)) {
  console.log(i);
}
```

或者在对象的原型上添加接口；

```javascript
function* gen() {
  for (const key in this) {
    if (this.hasOwnProperty(key)) {
      const element = this[key];
      yield element;
    }
  }
}
Object.prototype[Symbol.iterator] = gen;
let obj = new Object({
  a: 1,
  b: 2
});
for (const i of obj) {
  console.log(i);
}
// 1
// 2
[...obj];
// 1
// 2
```

上述代码将 Generator 函数部署在对象的原型上，除了`for of`外，`... | Array.from | 解构赋值`等可以调用内部遍历器接口的方法，都可以直接使用。

### Generator.prototype.throw

在 Iterator 那篇里面也提到了，遍历器对象可以有`next | return | throw`方法，当时没有细讲`throw`方法，因为这个方法抛出的异常可以在 Generator 函数体内捕获；

```javascript
function* gen() {
  try {
    yield '1';
  } catch (e) {
    console.log(e, '2');
  }
}
let g = gen();
g.next();
// {value: "1", done: false}
g.throw('??');
// ?? 2
// {value: undefined, done: true}
g.throw('?');
// Uncaught ?
try {
  g.throw('???');
} catch (e) {
  console.log(e, '3');
}
// ??? 3
```

可以看到在 Generator 函数体内，有一个`try catch`块，用于捕获对象`g`，或是由 Generator 函数内部抛出的异常（毕竟这才是`try catch`块的原来作用）；下面来分析一下这段代码；

- 对象`g`调用`next`方法时，执行到了`yield`语句，返回对象`{value: "1", done: false}`；
- 接着调用了`throw`方法，这个方法会向`gen`函数体内抛出一个异常，此时函数体内正好有`try catch`块，就被函数内部的`catch`函数捕获了，可以看到`throw`方法传入了`'??'`参数，执行打印语句，打印出参数和字符串`'2'`，并返回了对象`{value: undefined, done: true}`，因为已经没有`yield`表达式了，所以此时`gen`函数已经执行完了；
- 再次调用`throw`方法并传入参数`'?'`，函数已经执行完毕，向函数体内抛异常就不会被内部捕获了，所以就冒泡形式抛出了函数体外，外部也没有捕获函数，所以会报未捕获的异常；
- 最后在函数体外写了一个`try catch`块，有了捕获函数，就表现为正常的捕获，但这时候已经和普通抛出异常没什么区别了；

总的来说遍历器对象的`throw`函数，本质是向 Generator 函数内部抛出异常，由函数内部先进行捕获，如果没有被函数体内捕获，那么函数就会退出执行，并将异常抛出到函数体外部，这就很像事件的冒泡执行，一层层由里到外的冒泡，只是被捕获了之后就不会继续冒泡了；

还有一个点就是调用了`throw`方法后，如果异常被函数内部捕获了，会自动执行一次`next`方法；

```javascript
function* gen() {
  try {
    yield '1';
    // throw new Error('!!');
  } catch (e) {
    console.log(e, '2');
  }
  yield '2';
  yield '3';
}
let g = gen();
g.next();
// {value: "1", done: false}
g.throw('????');
// ???? 2
// {value: "2", done: false}
g.next();
// {value: "3", done: false}
g.next();
// {value: undefined, done: true}
```

可以看到在第一次调用`throw`方法时，不仅执行了捕获函数`catch`（执行`console.log`语句），还执行了第二个`yield`表达式，返回了`{value: "2", done: false}`对象，这样可以不让错误影响下一个`yield`的执行；

这样设计的好处是，我们在 Generator 函数体外部向函数体内部抛出异常，由函数内部捕获异常，异常被处理后不会影响函数接下来的处理，还能继续在外部调用`next`方法继续执行 Generator 函数；

依靠这种方式，我们在写 Generator 函数时，可以把大部分`yield`表达式放在`try catch`块里，这样只需要用一个捕获函数来捕获错误；

> 值得注意的是，从异常抛出的角度来说，任何函数体内的异常，如果没有在内部被捕获，都会导致异常冒泡到函数体外，并且会导致函数中断执行；

所以如果我们在`try catch`块里写了很多`yield`，其实也只能捕获第一次抛出的异常，如果在后续代码继续执行`throw`方法，就会导致函数中断执行从而退出；

```javascript
function* gen() {
  try {
    yield '1';
    yield '4';
    yield '5';
    yield '6';
    yield '7';
    // throw new Error('!!');
  } catch (e) {
    console.log(e, '2');
  }
  yield '2';
  yield '3';
}
let g = gen();
g.next();
// {value: "1", done: false}
g.throw('e');
// e 2
// {value: "2", done: false}
g.throw('?');
// Uncaught ?
g.next();
// {value: undefined, done: true}
```

上面的代码验证了当函数体内部没有捕获到异常时，函数就会中断执行并退出了；

### Generator.prototype.return

和 Iterator 一样，Generator 函数返回的遍历器对象还有一个`return`方法，可以返回给定的值并且会终止遍历 Generator 函数。

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
let g = gen();
g.next();
// {value: 1, done: false}
g.return('?');
// {value: "?", done: true}
g.next();
// {value: undefined, done: true}
```

可以看到如果`return`方法带参数了，返回的对象的`value`属性值就是参数值；如果不带参数就是`undefined`；另外也可以看到调用了这个方法后函数就终止执行了；

关于`return`方法，还有一点要说明的，如果在 Generator 函数内部有`try finally`代码块，并且函数正在执行`try`的代码块，那么`return`方法会导致立刻进入`finally`代码块，执行后才会终止函数，来看个例子：

```javascript
function* gen() {
  yield 1;
  try {
    yield 2;
    yield 3;
  } catch (error) {
  } finally {
    yield 4;
  }
  yield 5;
}
let g = gen();
g.next();
// {value: 1, done: false}
g.next();
// {value: 2, done: false}
g.return('?');
// {value: 4, done: false}
g.next();
// {value: "?", done: true}
```

可以看到，函数运行到`yield 2`时，调用了`return`方法，此时函数跳过`yield 3`，进入到`finally`代码块，并执行了`yield 4`（如果代码块后续还有`yield`方法，还可以继续调用`next`方法去执行），此时调用`next`方法会返回`return`方法的参数`'?'`，作为返回对象的`value`属性值，且此时函数已经执行完毕。

#### next | throw | return 的共同点

这三个方法本质都是让 Generator 函数继续执行，并且使用不同的语句替换`yield`表达式；

怎么理解呢，`next`方法将表达式替换为一个值，因为前面在讲`next`方法时提到过，如果给`next`方法传入了参数，那么上一个`yield`表达式会被替换为参数值，不传参时（可以当做是传入了`undefined`，表达式的值就是`undefined`，很合理；

`throw`方法把`yield`表达式替换成一个函数内部的`throw`语句；

`return`方法把`yield`表达式替换成`return`语句；

这样记是不是方便多了= =；

### yield\* 表达式

语法上是在`yield`关键字后面加了个`*`符号，表示批量执行一个 Generator 函数；我们知道只有在 Generator 函数内部才能使用`yield`表达式，所以`yield*`也只能在 Generator 函数内部使用，那就意味着这个表示是用来在一个 Generator 函数内部执行另一个 Generator 函数的，来看个例子：

```javascript
function* gen() {
  yield 1;
  yield 1;
  yield 1;
  yield 1;
}
function* gen2() {
  yield 2;
  yield* gen();
  yield 3;
}
let g = gen2();
for (const i of g) {
  console.log(i);
}

// 如果不用yield*
function* gen() {
  yield 1;
  yield 1;
  yield 1;
  yield 1;
}
function* gen2() {
  yield 2;
  for (const i of gen()) {
    yield i;
  }
  yield 3;
}
let g = gen2();
for (const i of g) {
  console.log(i);
}
```

其实从上面的代码可以看出来，`yield*`语法只是一种遍历的简写形式，其实看起来和`for of`是等价的，但是在`gen`函数有返回值时，就不等价了，因为用`for of`循环获取不到`return`的值，用`yield*`是可以的，`yield\*`整个表达式的值就是紧跟着的表达式的返回值；

```javascript
function* gen() {
  yield 1;
  yield 1;
  return 10;
}
function* gen2() {
  yield 2;
  console.log(yield* gen());
  // 10
  yield 3;
}
let g = gen2();
for (const i of g) {
  console.log(i);
}
```

从代码中还能看出其实`yield*`后面紧跟的表达式是一个遍历器对象，我们试试在具有`Iterator`接口的数据结构上使用`yield*`；

```javascript
function* gen() {
  yield* [1, 2, 3];
  yield* '??';
}
for (const i of gen()) {
  console.log(i);
}
// 1
// 2
// 3
// ?
// ?
```

利用这个特性我们可以将嵌套数组展开成一维数组；

```javascript
function* flatten(ary) {
  for (const i of ary) {
    if (Array.isArray(i)) {
      yield* flatten(i);
    } else {
      yield i;
    }
  }
}
[...flatten([1, 2, [3, [4, [5]]]])];
// [1, 2, 3, 4, 5]
```

### Generator 函数作为对象的属性

在语法上，Generator 函数作为对象属性时可以简写：

```javascript
let obj = {
  *gen() {
    // do something
  }
};
```

### Generator 函数的 this

Generator 函数总是返回遍历器对象，语法规定这个遍历器是 Generator 函数的实例，也继承了原型链上的方法，是不是感觉 Generator 函数很像是类的构造函数，区别就在于 Generator 函数返回的是遍历器对象，而不是普通构造函数的`this`；

```javascript
function* gen() {
  this.a = 1;
}
gen.prototype.x = () => console.log('?');
let g = gen();
g.x();
// ?
g.a;
// undefine
```

可以看到 Generator 函数返回的遍历器对象`g`，可以访问原型链上的`x`函数，却不能访问函数`this`的属性，说明了遍历器对象不是`gen`函数的`this`；另外对`gen`函数使用`new`运算符会报错，因为 Generator 函数本来就不是构造函数，下面写了一种方法可以获取到 Generator 函数实例对象的方法；

```javascript
function* gen() {
  this.a = 1;
  yield (this.b = 2);
}
function G() {
  return gen.call(gen.prototype);
}
let g = new G();
g.next();
// {value: 2, done: false}
g.a;
// 1
Reflect.ownKeys(g);
// []
gen.prototype;
// Generator {a: 1, b: 2}
```

就是调用`call`函数，让`gen`函数执行时，函数内部的`this`绑定到`gen`函数的原型上，所以其实相当于访问的是原型上的的`a | b`属性，暂时没想到什么场景会使用到。

### Generator 函数的应用

#### 将异步操作同步化表达

将异步操作写在`yield`表达式里，等到调用`next`方法后再继续执行，这种方式就不需要写回调函数了，因为一般我们写回调函数就是想让当前操作结束后，再去调用回调函数，而异步操作的后续操作可以直接写在`yield`表达式后面。

```javascript
function* loadUI(loadDataAsync) {
  showLoading();
  yield loadDataAsync();
  hideLoading();
}
let loader = loadUI(getData);
// 显示loading的UI，并执行异步获取数据的函数
loader.next();
// 隐藏loading
loader.next();
```

如果上面的代码要改写成回调函数的形式，只能在`loadDataAsync`函数里执行`hideLoading`函数了；明显不如上面的代码条理清晰；一个更为实际的例子：

```javascript
function loadData() {
  let xhr = new XMLHttpRequest();
  xhr.open('get', 'https://cdn.bootcss.com/jquery/3.4.1/jquery.slim.min.js');
  xhr.send();
  xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
      loader.next(xhr.responseText);
    }
  };
}
const showLoading = () => {
  console.log('?');
};
const hideLoading = r => {
  console.log(r);
};
function* loadUI(loadDataAsync) {
  showLoading();
  let r = yield loadDataAsync();
  hideLoading(r);
}
let loader = loadUI(loadData);
// 显示loading的UI，并执行异步获取数据的函数
loader.next();
// 隐藏loading
// loader.next();
```

这段代码随意定义了一个`loadData`函数，用于发送`ajax`请求，并且把`loadUI`函数继续执行的权力交给了实际的异步函数，当异步函数完成自己的功能之后，会调用遍历器的`next`方法，然后就会继续执行`loadUI`函数的后续步骤；并且由于函数是异步的，所以在第一次调用`loader.next`方法时，不会阻塞进程，后续的代码会接着执行；其实这样写的主要目的还是让代码结构看起来清晰，看起来就像是同步函数，按顺序挨个执行；

#### 部署 Iterator 接口

其实在 Iterator 那篇里提过如何在对象等数据结构上部署`Iterator`接口，这里再用 Generator 函数来部署一次；

```javascript
function* iterObj(obj) {
  let keys = Reflect.ownKeys(obj);
  for (let i of keys) {
    yield [i, obj[i]];
  }
}
let obj = {
  a: 1,
  b: 2
};
for (let i of iterObj(obj)) {
  console.log(i);
}
// ["a", 1]
// ["b", 2]
```

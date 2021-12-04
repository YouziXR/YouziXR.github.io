# ES6 新特性学习第九篇

## Iterator | for of

ES6 中新增了`Map | Set`两种表示集合的数据结构，这样就有了`Object | Array | Map | Set`四种结构可以来表示一组数据，为了统一使用一种遍历访问的接口机制来访问这些数据结构，而创建了`Iterator`（遍历器），只要数据结构部署了 Iterator 接口，就可以完成遍历操作；Iterator 作用有三个：提供遍历接口；使数据结构成员能够按照某种顺序进行排列；ES6 中可以使用`for of`来遍历成员，而 Iterator 也主要是为了给`for of`消费。

### Iterator

#### Iterator 遍历过程

1. 创建指针对象，指向起始位置；可以认为 Iterator 对象其实就是一个指向成员的指针对象；
2. 调用指针对象的`next`方法，返回当前指针指向成员的信息；
3. 重复 2，直到指针指向结束位置；

指针每次返回的成员信息是一个有`value | done`两个属性的对象，`value`就是成员的值，`done`是布尔值，标志着是否遍历结束；下面的代码模拟了一个 Iterator 类：

```javascript
class iterator {
  constructor(arg) {
    this.arg = arg;
    this.length = arg.length;
    this.index = 0;
  }
  next() {
    return this.index < this.length
      ? {
          value: this.arg[this.index++],
          done: false
        }
      : {
          value: undefined,
          done: true
        };
  }
}
let it = new iterator([1, 2]);
it.next(); // {value: 1, done: false}
it.next(); // {value: 2, done: false}
it.next(); // {value: undefined, done: true}
```

#### 数据结构中默认的 Iterator 接口

前面也提到过，Iterator 接口的目的是为数据结构提供统一的访问机制，即使用`for of`就可以遍历数据结构，代码运行时会主动寻找 Iterator 接口；而默认的 Iterator 接口部署在数据结构的`Symbol.iterator`属性中，可以认为数据结构有这个属性就是可遍历的；关于这个属性，这是某些对象内置的 Symbol 值，指向了语言内部使用的`iterator`方法，访问这个属性相当于调用了对象内部的`iterator`方法。

如下的数据结构原生就具有 Iterator 接口：

- Array
- Map
- Set
- String
- TypedArray
- arguments 对象
- NodeList 对象

上述数据结构由于原生就有`Iterator`接口，所以不需要自己单独写遍历器生成函数，可以直接用`for of`直接遍历；如果想在其他数据结构上也用`for of`遍历，则需要自己部署`Iterator`接口。

其实写到这里我自己也有点好奇，为什么相似的数据结构`Map | Object`在迭代器的表现上是不一致的，查了 MDN 之后看出一个很重要的点就是`Map`的`key`值是有序的，但`Object`不是，所以就需要手动去确定遍历的顺序，看下面的例子，在对象上添加`Symbol.iterator`属性，并部署遍历器生成方法（或者部署在原型链上）；

> 其实 ES6 规范后的版本，`Object`保留了字符串和`Symbol`的`key`的创建顺序，所以如果遍历只有字符串和`Symbol`作为`key`的对象，是按插入顺序遍历的；

```javascript
class ObjIterator {
  constructor(obj) {
    this.obj = obj;
    this.index = 0;
    this.keys = Reflect.ownKeys(obj);
  }
  [Symbol.iterator]() {
    return this;
  }
  next() {
    if (this.index < this.keys.length) {
      let key = this.keys[this.index];
      let val = this.obj[key];
      this.index++;
      return {
        value: val,
        done: false
      };
    } else {
      return {
        value: undefined,
        done: true
      };
    }
  }
}
let obj = {
  a: 1,
  b: 2
};
let o = new ObjIterator(obj);
for (const i of o) {
  console.log(i);
}
```

也可以在数据结构原型上添加遍历器生成方法：

```javascript
const iterator = function() {
  // this: object
  const keys = Reflect.ownKeys(this);
  const length = keys.length;
  let index = 0;
  const that = this;
  return {
    next() {
      let key = keys[index];
      let val = that[key];
      if (index < length) {
        index++;
        return {
          done: false,
          value: val
        };
      } else {
        return {
          done: true,
          value: undefined
        };
      }
    }
  };
};
Object.prototype[Symbol.iterator] = iterator;
let o = new Object({ a: 1, b: 2 });
for (let i of o) {
  console.log(i);
}
// 1
// 2
```

如果想在类数组对象上添加`Iterator`接口（其实类数组对象本身就具有遍历器，可以直接访问），可以通过`Array.prototype[Symbol.iterator]`来访问到数组的遍历器，直接给类数组对象的原型赋值即可；

这里要注意区分一个概念，遍历器接口和遍历器对象；遍历器接口我认为是一个遍历器对象的生成函数，是用来生成遍历器对象的，就如上面的代码那样，`iterator`是一个函数，用于生成一个带有`next`方法和其他属性的遍历器对象；而遍历器对象就是带有`next`方法的一个对象，我们也可以显式的去访问这个对象的`next`方法，如下面的例子：

```javascript
let aryLike = {
  0: 1,
  1: 2,
  2: 3,
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
let iter = aryLike[Symbol.iterator]();
iter.next();
// {value: 1, done: false}
iter.next();
// {value: 2, done: false}
iter.next();
// {value: 3, done: false}
iter.next();
// {value: undefined, done: true}
```

那像上面那样的代码，我们可以先取得遍历器对象，然后用`while | for`循环等方法去调用对象的`next`方法，直到返回对象的`done`为`true`就停止循环，以达到遍历的效果；

#### 调用`Iterator`接口的场景

最常见的就是`for of`命令，会调用`Symbol.iterator`方法，还有一些其他的：

1. 解构赋值

对数组和`Set`结构进行解构时，会默认调用`Iterator`接口；

2. 扩展运算符

扩展运算符也会调用`Iterator`接口，但对于对象来说只能通过`{...obj}`来调用扩展运算符，用`[...obj]`会报错的，这说明了其实对象并没有部署遍历器接口；只要部署了遍历器接口的对象，就可以用扩展运算符将这个对象转化为数组；

3. yield\*

`yield`后面跟一个可遍历的接口，会自动调用该结构的遍历器接口。

4. 其他

其实因为只要遍历了数组就会调用数组的遍历器接口，所以只要是数组作为参数的情况，就都调用了遍历器接口；

#### 字符串的遍历器接口

字符串原生部署了遍历器接口，原生的遍历器将字符串分割成单个字符并返回，效果类似于调用了`String.prototype.split`方法；当然也可以重写`String`原型上的`[Symbol.iterator]`属性以达到自定义遍历器行为的目的；

#### Iterator 接口与 Generator 函数

`Symbol.iterator`方法的最简单实现，是使用 Generator 生成器函数；

#### 遍历器对象的其他属性和方法

在上面的代码中我们很多都是自己部署了遍历器接口，返回的对象都包括了`next`方法，这个方法是必须的，除此之外还有`return | throw`方法是可选的；

`return`方法使用场景是当`for of`循环提前结束（可能是出错或者是遇到了`break`语句），就会自动调用`return`方法；通常，如果在遍历完成之前需要清理或者释放内存资源，就可以部署`return`方法，来看下面的例子；

```javascript
let ary = [1, 2, 3, 4];
ary[Symbol.iterator] = function() {
  return {
    next() {
      return {
        value: 2,
        done: false
      };
    },
    return() {
      console.log('come to an end');
      return { done: true };
    }
  };
};
for (let i of ary) {
  if (i > 1) {
    console.log('?');
    break;
  } else {
    console.log(i);
  }
}
// ?
// come to an end
for (let i of ary) {
  if (i > 1) {
    console.log('?');
    throw new Error('!');
  } else {
    console.log(i);
  }
}
// ?
// come to an end
// error
```

`throw`主要配合 Generator 函数一起用，一般的遍历器对象用不到这个方法；

#### for of 循环

这个一开始就提到过了，算是很常用的遍历方法了，本质就是调用了数据结构内部的`Symbol.iterator`方法；

### 返回遍历器对象的场景

在 ES6 中，三种数据结构`Array | Set | Map`都部署了以下三个方法，方法调用后会返回遍历器对象；

- `entries`用于遍历`[key, value]`组成的数组，数组的`key`是索引值，Set 的`key === value`，Map 就直接返回了，而且 Map 的`Iterator`接口默认就调用了`entries`方法；
- `keys`用于遍历所有`key`；
- `values`用于遍历所有`value`；

### 对比其他遍历方法

ES6 语法之后，JS 提供了挺多的遍历方法，针对不同的数据结构，一般常用的是原始的`for | while`循环等，但是写起来比较麻烦；针对数组提供了`forEach | map | filter`等方法，当然作用是不一样的，都能达到遍历的效果；针对对象还有`for in| keys`等方法，重点比较这几个的优缺点；

- `for | while`循环，自己要写比较多的代码去访问到数组成员，不如`forEach`简洁明了，当然好处就是自己能完全控制数组下标；
- `forEach`写法简洁，下标固定按照步长为 1 去访问每个数组成员，但是不好跳出循环，经常用的是`try catch`方法去捕获异常，强制跳出；
- `for in`这个方法其实本来是为对象提供的，用于访问对象的`key`，在数组对象上调用这个方法，访问的就是数组的下标（字符串形式，即`'0','1'`）；这个方法访问对象时会把对象原型链上的`key`也访问到，如果只想访问本对象的`key`一般用`keys`方法；
- `for of`语法简洁，可以手动`break | continue | return`，相对来说是比较通用的一个遍历方法了。

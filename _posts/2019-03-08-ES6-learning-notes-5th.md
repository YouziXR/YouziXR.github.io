# ES6新特性学习第五篇 #

## 前言 ##

好久没写了，前几天把以前的blog重新看了看，继续学习ES6。

## 第一篇前言 ##

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 原著:阮一峰 ") 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ "阮一峰ES6")

### Symbol

为了解决对象属性命名冲突的问题，引入了`Symbol`数据类型（其他6种：`undefined，null，Boolean，String，Number，Object`），标识了唯一的值；生成一个`Symbol`类型的值通过函数`let symbol = Symbol("args")`，注意`Symbol()`函数不是构造函数，所以不能用`new`，参数是对Symbol的描述，为了在控制台显示方便，以及转成字符串时好区分；另外如果参数是个对象，会调用对象的`toString`方法，转成字符串再生成值。

注意一点：即使`Symbol`的参数一样，变量也是不相等的。

`Symbol`可以显式转换成字符串或者布尔值，但不能转成数值，因此不能和其他类型的值进行运算。

#### Symbol的应用


- 作为对象的属性名：如果一个对象由很多模块构成，可能会出现同名属性，此时用`Symbol`就能防止`Key`被覆盖；两种写法。

        let symbol = Symbol()
        let o1 = {}
        o1[symbol] = '?'
        let o2 = {
            [symbol]: '??'
        }
        o1.symbol // undefined
        o1.symbol = '???'
        o1[symbol] // '?'
        o1['symbol'] // '???'

由上述代码可知，对象内部使用`Symbol`定义属性时，必须放在方括号中。

另外Symbol还可以用来定义一组常量，保证值唯一且不相等。

	const pattern = {
        PROD: Symbol('product'),
        DEV: Symbol('develop'),
        TEST: Symbol('test')
    }

##### 消除魔术字符串

我还是第一次听到这个说法，魔术字符串指：在代码中多次出现、与代码形成强耦合的某个具体字符串或者数值；所以应该尽量消除，换成有含义的变量。

    function func(args) {
        switch (key) {
            // 如果在很多位置都要使用这个字符串，建议保存在变量里
            case 'James': // 魔法串
            // do something
            break;
            case 'Marry': // 魔法串
            // ...
            break;
            default:
            break;
        }
    }

    let keys = {
        James: Symbol('James'),
        Marry: Symbol('Marry')
    }
    function func(args) {
        switch (args) {
            case keys.James:
                // do something
                break;
            case keys.Marry:
                // ...
                break;
            default:
                break;
        }
    }

##### Symbol不会被常规方法遍历

将Symbol作为属性名时，这个属性不会被`for in || for of || Object.keys() || Object.getOwnPropertyNames() || JSON.stringify()`遍历到，但并不意味着它是个私有属性，ES6新增方法`Object.getOwnPropertySymbols()`，可以获取对象的所有Symbol；另外`Reflect.ownKeys()`方法可以返回所有类型的`key`。

这个特性可以应用到定义一些**非私有，但只用于内部的方法**。

##### 全局Symbol

我们已经知道如何生成一个普通的`Symbol`值，但有时会希望重新使用同个Symbol（出于节约内存的考虑等），`Symbol.for(args)`方法会在全局作用域内搜索以参数命名的Symbol，有就返回这个值，没有就新建并返回；

所以`Symbol() || Symbol.for()`的区别就是是否在全局登记，例如调用`Symbol('foo')`30次会生成30个不同的Symbol，但调用`Symbol.for('foo')`30次只会返回1个Symbol；调用`Symbol.keyFor(symbol)`方法会返回一个已经登记的Symbol

    ss = Symbol.for('???')
    // Symbol(???)
    Symbol.keyFor(ss)
    // "???"

另外，登记的Symbol是在全局环境下的，可以在不同的`iframe`中取得。

### Set

一种新的数据结构，类似数组，但它的元素值是唯一的。

`Set()`是一个构造，直接`new`一个Set数据结构；还可以通过接受具有Iterator接口的数据结构作为参数，进行初始化赋值。

利用Set去重的方法：

    [...new Set(array)]
    [...new Set('abbbbbc')].join('')

注意：**向Set添加值的时候不会发生类型转换**

#### 实例属性和方法

- `Set.prototype.constructor`：构造函数
- `Set.prototype.size`：实例的元素总数
- `add(val)`：添加元素，返回Set
- `delete(val)`：删除元素，返回布尔值
- `has(val)`：判断值是否为Set的元素，返回布尔值
- `clear()`：清空
- `keys() || values() || entries() || forEach()`：和数组一样的

值得注意的是：`Set`结构的`key == value`，来看一个例子，在浏览器控制台操作的：

    for (let i of new Set("abcd").entries()) {
    console.log(i);
    }
    // VM716: 1(2)[("a", "a")];
    // VM716: 1(2)[("b", "b")];
    // VM716: 1(2)[("c", "c")];
    // VM716: 1(2)[("d", "d")];

    for (let [k, v] of new Set("abcd").entries()) {
    console.log(k == v);
    }
    // 4 VM723: 1 true

#### Set间接使用Array所有方法

由于`Set`和`Array`可以互相转换，所以数组方法基本上在`Set`都可以调用。



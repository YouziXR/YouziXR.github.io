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

由于`Set`和`Array`可以互相转换，所以数组方法基本上在`Set`都可以调用。套用以下模板：

    let set = new Set([1,2,3])
    set = new Set([...set].arrayMethod(...))
	// 或者使用Array.from(set, methdod...)

Set交并补集的实现：

	let a = new Set([1, 2, 3]);
	let b = new Set([4, 3, 2]);
	
	// 并集
	let union = new Set([...a, ...b]);
	// Set {1, 2, 3, 4}
	
	// 交集
	let intersect = new Set([...a].filter(x => b.has(x)));
	// set {2, 3}
	
	// 差集
	let difference = new Set([...a].filter(x => !b.has(x)));
	// Set {1}

### WeakSet

WeakSet结构是Set结构的特殊化，它只能存放对象，且存放的都是弱引用。即垃圾回收机制不考虑在WeakSet中对某个对象的引用，如果这个对象不再被其他对象引用，那么垃圾回收机制会自动回收这个对象的内存，不考虑这个对象还存放在WeakSet中。

垃圾回收机制依赖引用计数，如果引用次数不为0，就不会释放内存占用，有些情况下会忘记取消引用，导致内存一直被占用，就可能引发内存泄漏。引入WeakSet就是为了临时存放一组对象，只要存放的对象在外部释放，那它在WeakSet里的引用就无效了，基于此，WeakSet是不可遍历的。

实例对象有三个方法：

- `add(val)`：添加新对象
- `delete(val)`：清楚对象
- `has(val)`：是否存在某个对象

### Map

JS中的对象是`key-value`的Hash结构，但`key`的数据类型只能是字符串；所以为了便于使用引入了`Map`结构，Map类似于对象，也是`key-value`的集合，但`key`的数据类型不仅是字符串，可以是各种类型的值。

Map自身是一个构造函数，并且可以接受一个数组作为参数；

	const map = new Map([
	  ['name', '张三'],
	  ['title', 'Author']
	]);
	
	map.size // 2
	map.has('name') // true
	map.get('name') // "张三"
	map.has('title') // true
	map.get('title') // "Author"

	// 实际上执行的是下面的算法
	const items = [
	  ['name', '张三'],
	  ['title', 'Author']
	];
	
	const map = new Map();
	
	items.forEach(
	  ([key, value]) => map.set(key, value)
	);

任何具有Iterator接口、且每个成员都是双元素数组的数据结构都可以当做Map构造函数的参数；Set和Map都可以作为参数来生成新的Map。

当Map的键是简单类型的值（数字、字符串、布尔值），只要值严格相等，Map就视为一个键；如果键是对象，只要对象指向的内存地址不一样，就会视为两个键。

#### 实例属性和方法

- size：返回Map成员总数。
- set(key, val)：添加成员，返回整个Map结构，可以采用链式写法。
- get(key)：获取对应的val，不存在则返回undefined。
- has(key)：判断是否key是否在Map对象中。
- delete(key)：删除key，返回是否删除成功。
- clear()：清空。无返回值。
- keys()：返回键名的遍历器。
- values()：返回键值的遍历器。
- entries()：返回所有成员的遍历器。
- forEach()：遍历Map的成员。

特别注意：Map的遍历顺序就插入的顺序；`for [key, val] of map`这个方法遍历Map与用entries是一样的。

和Set一样，可以使用扩展运算符将Map结构转换成数组结构，就可以使用数组的各种方法，但是注意Map结构有特殊性。

#### Map数据结构转换

- Map<->数组：扩展运算符和Map构造函数。
- Map<->对象：我觉得这个不用说了，用`for...of`结构取出成员组合成对象或者Map结构就行。
- Map<->JSON：Map转JSON，可以先转成对象或者数组，再调用`JSON.stringify()`；JSON转Map，先调用`JSON.parse()`将JSON转成对象，再调用对象转Map方法。

### WeakMap

该结构只接受对象作为键。其他的和WeakSet差不多。典型应用有，向DOM上添加数据，很多时候DOM元素都是很占用内存的。

特别注意：WeakMap结构只有键是弱引用，值还是正常引用的。

只有4个方法可用：`get，set，has，delete`

来看一个用`WeakMap`添加私有属性的例子。

    const _counter = new WeakMap();
    const _action = new WeakMap();
    class Countdown {
        constructor(counter, action) {
            _counter.set(this, counter);
            _action.set(this, action);
        }
        dec() {
            let counter = _counter.get(this);
            if (counter < 1) return;
            counter--;
            _counter.set(this, counter);
            if (counter === 0) {
                _action.get(this)();
            }
        }
    }
    const c = new Countdown(2, () => console.log('DONE'));
    c.dec()
    c.dec()
    // DONE


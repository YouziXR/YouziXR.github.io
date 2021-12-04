# ES6新特性学习第四篇 #

## 前言 ##

承接上一篇，本篇从对象扩展开始学习。

## 第一篇前言 ##

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 原著:阮一峰 ") 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ "阮一峰ES6")

## 对象的扩展 ##

### 属性的简洁表示法 ###

ES6允许直接写入变量和函数，作为对象的属性和方法。此时属性名为变量名，属性值为变量的值。

	function f(x, y) {
	  return {x, y};
	}
	// 等同于
	function f(x, y) {
	  return {x: x, y: y};
	}
	f(1, 2) // Object {x: 1, y: 2}

除了属性，方法也可以简写。

	const o = {
	  method() {
	    return "Hello!";
	  }
	};
	// 等同于
	const o = {
	  method: function() {
	    return "Hello!";
	  }
	};

需要注意的是，简介写法的属性名总是字符串。

### 属性名表达式 ###

JS定义对象的属性，有两种方法。

	// 方法一
	obj.foo = true;
	// 方法二
	obj['a' + 'bc'] = 123;

一是直接用标识符作为属性名，二是用表达式作为属性名，此时要把表达式放在括号里。

如果使用字面量定义对象，ES5中只能用标识符定义属性，在ES6里允许用表达式作为对象的属性名。

	var obj = {
	  foo: true,
	  abc: 123
	};
	let propKey = 'foo';
	let obj = {
	  [propKey]: true,
	  ['a' + 'bc']: 123
	};
	let lastWord = 'last word';
	const a = {
	  'first word': 'hello',
	  [lastWord]: 'world'
	};
	a['first word'] // "hello"
	a[lastWord] // "world"
	a['last word'] // "world"

在方括号内的都被当做表达式处理。属性名表达式如果是一个对象，默认情况会自动将对象转化成字符串`[object Object]`。

	const keyA = {a: 1};
	const keyB = {b: 2};
	const myObject = {
	  [keyA]: 'valueA',
	  [keyB]: 'valueB'
	};
	myObject // Object {[object Object]: "valueB"}

由于`keyA ,keyB`都会转换成`[object Object]`字符串，所以后一个值会把前一个值覆盖，只会得到一个属性`valueB`。

### 方法的name属性 ###

函数`name`属性，返回函数名。对象方法也是函数，因此也有这个属性。这方法真的有用吗？？？首先要访问到这个函数，然后访问这个函数的属性`name`，有什么意义吗？

如果方法对象使用取值函数`getter, setter`，那么`name`属性不在该方法上，而是该方法的属性描述对象`get, set`属性上。

	const obj = {
	  get foo() {},
	  set foo(x) {}
	};
	obj.foo.name
	// TypeError: Cannot read property 'name' of undefined
	const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');
	descriptor.get.name // "get foo"
	descriptor.set.name // "set foo"

特殊情况和函数扩展那章的`name`属性一样，`bind`方法创造的函数会在原函数前加`bound`，`Function`构造函数创造的函数会加上`anonymous`。

	(new Function()).name // "anonymous"
	var doSomething = function() {
	  // ...
	};
	doSomething.bind().name // "bound doSomething"

如果对象方法是一个`Symbol`，那么`name`返回这个`Symbol`的描述。

	const key1 = Symbol('description');
	const key2 = Symbol();
	let obj = {
	  [key1]() {},
	  [key2]() {},
	};
	obj[key1].name // "[description]"
	obj[key2].name // ""

### Object.is() ###

ES5相等运算符和严格相等运算符。相等会进行数组类型转换，后者的`NaN`不等于自身，以及`+0 等于 -0`。在ES6中提出"same-value equality"算法，这个方法就是用来比较两个值是否严格相等，和严格相等运算符行为基本一致，不同之处是`NaN`等于自身，`-0 不等于 +0`

### Object.assign() ###

#### 基本用法 ####

`Object.assign`方法用于合并对象，将source的可枚举属性复制到target。

方法的第一个参数是target，后面的参数都是source。

	const target = { a: 1 };
	const source1 = { b: 2 };
	const source2 = { c: 3 };
	Object.assign(target, source1, source2);
	target // {a:1, b:2, c:3}

如果目标对象与源对象有同名属性，或者多个源对象有同名属性，则后面的属性会覆盖前面的属性。

	const target = { a: 1, b: 1 };
	const source1 = { b: 2, c: 2 };
	const source2 = { c: 3 };
	Object.assign(target, source1, source2);
	target // {a:1, b:2, c:3}

如果只有一个参数，方法会直接返回该参数。

	const obj = {a: 1};
	Object.assign(obj) === obj // true

如果该参数不是对象，则会转成对象然后返回。

	typeof Object.assign(2) // "object"

但是`undefined, null`无法转成对象，所以如果作为参数就会报错。

	Object.assign(undefined) // 报错
	Object.assign(null) // 报错

非对象参数出现在源对象的位置，那么处理规则有些不同，首先，这些参数都会转换成对象，如果无法转成对象就会跳过，也就是说如果`undefined, null`不在首参数，就不会报错。

	let obj = {a: 1};
	Object.assign(obj, undefined) === obj // true
	Object.assign(obj, null) === obj // true

其他类型的值（即数值、字符串和布尔值）不在首参数，也不会报错。但是，除了字符串会以数组形式，拷贝入目标对象，其他值都不会产生效果。这是因为只有`String`的包装对象会产生可枚举的属性，这些属性就会被拷贝。

	const v1 = 'abc';
	const v2 = true;
	const v3 = 10;
	const obj = Object.assign({}, v1, v2, v3);
	console.log(obj); // { "0": "a", "1": "b", "2": "c" }

将布尔值，数值，字符串分别转化成对应的包装对象，可以看到它们的原始值都在包装对象内部属性`[[PrimitiveValue]]`，这个属性不会被`Object.assign`拷贝。只有String的包装对象会产生可枚举的属性。

	Object(true) // {[[PrimitiveValue]]: true}
	Object(10)  //  {[[PrimitiveValue]]: 10}
	Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}

拷贝属性是有限制的，只拷贝源对象的自身属性，不拷贝集成属性，也不拷贝不可枚举属性（`enumerable: false`）。属性名为Symbol值的属性也会被拷贝。

#### 注意点 ####

##### 浅拷贝 #####

`Object.assign`方法实行的是浅拷贝，不是深拷贝，如果source某个属性值是对象，那么目标对象拷贝得到的是对象的引用。

##### 同名属性替换 #####

前面提过的，同名属性会覆盖。
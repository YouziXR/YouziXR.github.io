# ES6的一些新特性的学习 #
## let&const ##
### 块级作用域block ###
`let`只在块级作用域中有效

	{	let x = 1;	}
	x // ReferenceError

	var a = [];
	for (var i = 0; i < 10; i++) {
	  a[i] = function () {
 	   console.log(i);
 	 };
	}
	a[6](); // 10

	var a = [];
	for (let i = 0; i < 10; i++) {
  		a[i] = function () {
   	 	console.log(i);
 		};
	}
	a[6](); // 6
	循环语句部分是一个父作用域，而循环体内部是一个单独的子作用域。
	for (let i = 0; i < 3; i++) {
  		let i = 'abc';
 		console.log(i);
	}
	// abc
	// abc
	// abc
	在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。
	if (true) {
	  // TDZ开始
	  tmp = 'abc'; // ReferenceError
	  console.log(tmp); // ReferenceError

	  let tmp; // TDZ结束
	  console.log(tmp); // undefined

	  tmp = 123;
	  console.log(tmp); // 123
	}
### const ###
和`let`一样是只在块级域中有效，且不是变量的值不可改动，是变量指向的内存地址不得改动。JS中的简单数据类型包括数值，字符串，布尔值，值保存在变量指向的内存地址，等同于常量。

对象和数组变量保存的只是内存地址，`const`保证指向的内存地址不变，但其中的数据是可变的。

	const a = [];
	a.push('Hello'); // 可执行
	a.length = 0;    // 可执行
	a = ['Dave'];    // 报错
## 解构赋值 ##
### 数组解构赋值 ###

	let [ , , third] = ["foo", "bar", "baz"];
	third // "baz"

	let [x, , y] = [1, 2, 3];
	x // 1
	y // 3

	let [head, ...tail] = [1, 2, 3, 4];
	head // 1
	tail // [2, 3, 4]

	let [x, y, ...z] = ['a'];
	x // "a"
	y // undefined
	z // []
如果解构不成功，变量的值就等于`undefined`。

事实上，只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值。

### 默认值 ###
	let [foo = true] = [];
	foo // true

	let [x, y = 'b'] = ['a']; // x='a', y='b'
	let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'

ES6 内部使用严格相等运算符（`===`），判断一个位置是否有值。所以，如果一个数组成员不严格等于`undefine`，默认值是不会生效的。

	let [x = 1] = [undefined];
	x // 1

	let [x = 1] = [null];
	x // null

如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

	function f() {
	  console.log('aaa');
	}

	let [x = f()] = [1];

上面代码中，因为x能取到值，所以函数f根本不会执行。
### 对象解构赋值 ###
数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值。

对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。

	let { foo: baz } = { foo: "aaa", bar: "bbb" };
	baz // "aaa"
	foo // error: foo is not defined

上面代码中，`foo`是匹配的模式，`baz`才是变量。真正被赋值的是变量`baz`，而不是模式`foo`。

	var node = {
	  loc: {
	    start: {
	      line: 1,
	      column: 5
	    }
	  }
	};

	var { loc: { start: { line }} } = node;
	line // 1
	loc  // error: loc is undefined
	start // error: start is undefined

上面代码中，只有`line`是变量，`loc`和`start`都是模式，不会被赋值。

如果要将一个已经声明的变量用于解构赋值，必须非常小心。

	// 错误的写法
	let x;
	{x} = {x: 1};
	// SyntaxError: syntax error
	
	// 正确的写法
	({x} = {x: 1});
	// JavaScript引擎会将{x}理解成一个代码块

对象的解构赋值，可以很方便地将现有对象的方法，赋值到某个变量。

	let { log, sin, cos } = Math;

数组arr的0键对应的值是1，[arr.length - 1]就是2键，对应的值是3

	let arr = [1, 2, 3];
	let {0 : first, [arr.length - 1] : last} = arr;
	first // 1
	last // 3

### 字符串解构 ###
	
	const [a, b, c, d, e] = 'hello';
	a // "h"
	b // "e"
	c // "l"
	d // "l"
	e // "o"

### 其他数据形式 ###

解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

	let {toString: s} = 123;
	s === Number.prototype.toString // true
	
	let {toString: s} = true;
	s === Boolean.prototype.toString // true
	
	// 上面代码中，数值和布尔值的包装对象都有toString属性，因此变量s都能取到值。

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于undefined和null无法转为对象，所以对它们进行解构赋值，都会报错。

# ES6的一些新特性的学习 #

## 前言 ##

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 (原著:阮一峰) ")

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


在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

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

##  解构赋值  ##

### 数组解构赋值  ###

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

### 函数参数的解构赋值 ###

	function add([x, y]) {
		return x + y;
	}
	add([1, 2]); // 3

	[[1, 2], [3, 4]].map(([a, b]) => a + b);
	// [ 3, 7 ]

#### 函数参数解构的默认值 ####

例子1

	function move({x = 0, y = 0} = {}) {
	    return [x, y];
	}
	console.log(move({x: 3, y: 8}));// [3, 8]
	//说明：传递了实参，所以函数默认值{}不生效，对实参{x: 3, y: 8}解构赋值，对应的属性都有值，所以解构赋值的默认值不生效，所以x = 3，y = 8；
	
	console.log(move({x: 3}));// [3, 0]
	//说明：传递了实参，所以函数默认值{}不生效，对实参{x: 3}解构赋值，对应的属性x有值，y没有值，所以解构赋值的默认值y生效,x不生效，所以x = 3，y = 0；

	console.log(move({}));// [0, 0]
	//说明：传递了实参{}，所以函数默认值{}不生效，对实参{}解构赋值，对应的属性都没有值，所以解构赋值的默认值生效，所以x = 0，y = 0；

	console.log(move());// [0, 0]
	//说明：没有传递实参，所以函数默认值{}生效，对实参{}解构赋值，对应的属性都没有值，所以解构赋值的默认值生效，所以x = 0，y = 0；

例子2

	function move({x, y} = { x: 0, y: 0 }) {
       return [x, y];
    }
    console.log(move({x: 3, y: 8})); // [3, 8]
	//说明：传递了实参，所以函数默认值{x: 0, y: 0}不生效，对实参{x: 3, y: 8}解构赋值，对应的属性都有值，所以解构赋值的默认值不生效，所以x = 3，y = 8；
	
	console.log(move({x: 3})); // [3, undefined]
	//说明：传递了实参，所以函数默认值{x: 0, y: 0}不生效，对实参{x: 3}解构赋值，对应的属性x有值，y没有值，所以x=3,因为本例没有解构赋值的默认值，所以y就是undefined；

	console.log(move({})); // [undefined, undefined]
	//说明：传递了实参{}，所以函数默认值{x: 0, y: 0}不生效，对实参{}解构赋值，对应的属性没有值，所以x,y就是undefined；
	
	console.log(move()); // [0, 0]
	//说明：没有传递实参，所以函数默认值{x: 0, y: 0}生效，对实参{x: 0, y: 0}解构赋值，对应的属性有值，所以x,y就是0；

1. 判断调用过程中是否传递参数？若传递，用传递的实际参数为函数参数的默认值进行解构赋值；
2. 确定了解构赋值的对象后，看实参或函数参数默认值有没有变量中的对应属性，没有就解构默认值生效，否则解构默认值不生效。

### 使用圆括号的情况 ###

使用圆括号的情况只有一种：赋值语句的非模式部分，可以使用圆括号。

	[(b)] = [3]; // 正确
	({ p: (d) } = {}); // 正确
	[(parseInt.prop)] = [3]; // 正确

上面三行语句都可以正确执行，因为首先它们都是赋值语句，而不是声明语句；其次它们的圆括号都不属于模式的一部分。第一行语句中，模式是取数组的第一个成员，跟圆括号无关；第二行语句中，模式是p，而不是d；第三行语句与第一行语句的性质一致。

## 解构赋值 用途 ##

#### 交换变量的值 ####

	let x = 1;
	let y = 2;
	[x, y] = [y, x];

#### 函数返回多个值 ####

	// 返回一个数组
	function example() {
	  return [1, 2, 3];
	}
	let [a, b, c] = example();
	// 返回一个对象
	function example() {
	  return {
	    foo: 1,
	    bar: 2
	  };
	}
	let { foo, bar } = example();

#### 函数参数 ####

	// 参数是一组有次序的值
	function f([x, y, z]) { ... }
	f([1, 2, 3]);
	// 参数是一组无次序的值
	function f({x, y, z}) { ... }
	f({z: 3, y: 2, x: 1});

#### 提取JSON数据 ####

	let jsonData = {
	  id: 42,
	  status: "OK",
	  data: [867, 5309]
	};
	let { id, status, data: number } = jsonData;
	console.log(id, status, number);
	// 42, "OK", [867, 5309]

#### 函数参数的默认值 ####
	
	jQuery.ajax = function (url, {
	  async = true,
	  beforeSend = function () {},
	  cache = true,
	  complete = function () {},
	  crossDomain = false,
	  global = true,
	  // ... more config
	}) {
	  // ... do stuff
	};

#### 遍历Map结构 ####

部署了Iterator接口的对象，都可以用`for of`循环遍历。Map原生支持该接口，配合变量的解构赋值，获取`key`和`value`就很方便。

	var map = new Map();
	map.set('first', 'hello');
	map.set('second', 'world');
	
	for (let [key, value] of map) {
	  console.log(key + " is " + value);
	}
	// first is hello
	// second is world

	// 获取键名
	for (let [key] of map) {
	  // ...
	}
	// 获取键值
	for (let [,value] of map) {
	  // ...
	}

#### 输入模块的指定方法 ####
加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

	const { SourceMapConsumer, SourceNode } = require("source-map");

## 字符串拓展 ##

#### includes(), startsWith(), endsWith() ####

`includes()`：返回布尔值，表示是否找到了参数字符串。
`startsWith()`：返回布尔值，表示参数字符串是否在源字符串的头部。
`endsWith()`：返回布尔值，表示参数字符串是否在源字符串的尾部。
	
第二个参数，表示开始搜索的位置。

	var s = 'Hello world!';
	
	s.startsWith('world', 6) // true
	s.endsWith('Hello', 5) // true
	s.includes('Hello', 6) // false

`endsWith`的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束。

`repeat`方法返回一个新字符串，表示将原字符串重复n次。

	'x'.repeat(3) // "xxx"
	'hello'.repeat(2) // "hellohello"
	'na'.repeat(0) // ""

`padStart()`用于头部补全，`padEnd()`用于尾部补全。

    'x'.padStart(5, 'ab') // 'ababx'
    'x'.padStart(4, 'ab') // 'abax'
    'x'.padEnd(5, 'ab') // 'xabab'
    'x'.padEnd(4, 'ab') // 'xaba'

`padStart`的常见用途是为数值补全指定位数。
    
    '1'.padStart(10, '0') // "0000000001"
    '12'.padStart(10, '0') // "0000000012"
    '123456'.padStart(10, '0') // "0000123456"

另一个用途是提示字符串格式。
    
    '12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
    '09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"

### 模板字符串 ###

用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

    // 普通字符串
    `In JavaScript '\n' is a line-feed.`
    
    // 多行字符串
    `In JavaScript this is
     not legal.`
    
    console.log(`string text line 1
    string text line 2`);
    
    // 字符串中嵌入变量
    var name = "Bob", time = "today";
    `Hello ${name}, how are you ${time}?`

如果使用模板字符串表示多行字符串，所有的空格和缩进都会被保留在输出之中。

    $('#list').html(`
    <ul>
      <li>first</li>
      <li>second</li>
    </ul>
    `);

上面代码中，所有模板字符串的空格和换行，都是被保留的，比如`<ul>`标签前面会有一个换行。如果你不想要这个换行，可以使用`trim`方法消除它。

    $('#list').html(`
    <ul>
      <li>first</li>
      <li>second</li>
    </ul>
    `.trim());

模板字符串中嵌入变量，需要将变量名写在`${}`之中。

大括号内部可以放入任意的JavaScript表达式，可以进行运算，以及引用对象属性。

    var x = 1;
    var y = 2;
    
    `${x} + ${y} = ${x + y}`
    // "1 + 2 = 3"
    
    `${x} + ${y * 2} = ${x + y * 2}`
    // "1 + 4 = 5"
    
    var obj = {x: 1, y: 2};
    `${obj.x + obj.y}`
    // 3

### 标签模板 ###

模板标签是函数调用的一种特殊形式，标签指的是函数，后面模板字符串就是函数参数。

但如果模板字符串里有变量，会将字符串处理成多个参数，再调用函数。

	var a = 5;
	var b = 10;
	tag`Hello ${ a + b } world ${ a * b }`;
	// 等同于
	tag(['Hello ', ' world ', ''], 15, 50);

`tag`函数的第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，也就是说，变量替换只发生在数组的第一个成员与第二个成员之间、第二个成员与第三个成员之间，以此类推。

`tag`函数的其他参数，都是模板字符串各个变量被替换后的值。由于本例中，模板字符串含有两个变量，因此`tag`会接受到`value1`和`value2`两个参数。

模板字符串中有2个变量，说明字符串数组里应该有3个变量，所以字符串数组中最后加了一个`""`空串来补足。

	var total = 30;
	var msg = passthru`The total is ${total} (${total*1.05} with tax)`;
	function passthru(literals, ...values) {
	  var output = "";
	  for (var index = 0; index < values.length; index++) {
	    output += literals[index] + values[index];
	  }
	
	  output += literals[index];
	// 这里需要将字符串数组中的最后一个值添加进去
	// 字符串数组比参数多1
	  return output;
	}

#### 模板标签的应用 ####

过滤HTML字符串，防止用户输入恶意内容。

	var message =
	  SaferHTML`<p>${sender} has sent you a message.</p>`;
	
	function SaferHTML(templateData) {
	  var s = templateData[0];
	  for (var i = 1; i < arguments.length; i++) {
	    var arg = String(arguments[i]);
	
	    // Escape special characters in the substitution.
	    s += arg.replace(/&/g, "&amp;")
	            .replace(/</g, "&lt;")
	            .replace(/>/g, "&gt;");
	
	    // Don't escape special characters in the template.
	    s += templateData[i];
	  }
	  return s;
	}
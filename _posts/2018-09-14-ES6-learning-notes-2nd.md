# ES6的一些新特性的学习第二篇 #

## 前言 ##

本文承接上一篇学习笔记，从函数扩展开始学习。

### 上一篇前言： ###

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 (原著:阮一峰) ")

## 函数扩展 ##

### 参数默认值 ###

ES6前参数默认值的写法：

	function log(x, y) {
	  y = y || 'World';
	  console.log(x, y);
	}
	
	log('Hello') // Hello World
	log('Hello', 'China') // Hello China
	log('Hello', '') // Hello World

如果参数的布尔值为`false`，会被默认值覆盖，代码最后一行传入的空串也被默认值覆盖，这显然是不符合预期的。

为避免这个问题我们可以先判断参数是否被赋值，如果没有再用默认值覆盖。

	if (typeof y === 'undefined') {
	  y = 'World';
	}

ES6中为函数参数默认值提供了新的写法。

	function log(x, y = 'World') {
	  console.log(x, y);
	}
	
	log('Hello') // Hello World
	log('Hello', 'China') // Hello China
	log('Hello', '') // Hello

### 插播一条与或符号的笔记 ###

`expr1 && expr2` (逻辑与) 如果`expr1`能被转换为`false`，那么返回`expr1`；否则，返回`expr2`。因此，`&&`用于布尔值时，当操作数都为`true`时返回`true`；否则返回`false`.

`expr1 || expr2` (逻辑或) 如果`expr1`能被转换为`true`，那么返回`expr1`；否则，返回`expr2`。因此，`||`用于布尔值时，当任何一个操作数为`true`则返回`true`；如果操作数都是`false`则返回`false`。

上节代码中`y = y || 'world'`，如果`y`不为`false`返回`y`，否则返回`'world'`；

#### 关于短路求值 ####

	false && anything    // 被短路求值为false
	true || anything       // 被短路求值为true

上述表达式的anything部分不会被求值，所以这样做不会产生任何副作用。

### 函数默认值 ###

另一个例子：

	function Point(x = 0, y = 0) {
	  this.x = x;
	  this.y = y;
	}
	
	const p = new Point();
	p // { x: 0, y: 0 }

关于函数默认值，可以和第一篇学习笔记中的解构赋值相关章节结合，[https://youzixr.github.io/2018/09/07/ES6-learning-notes-1st/](https://youzixr.github.io/2018/09/07/ES6-learning-notes-1st/ "Learning notes 1st")

另外，参数默认值不是传值的，而是每次都重新计算默认值表达式的值。也就是说，参数默认值是惰性求值的。

	let x = 99;
	function foo(p = x + 1) {
	  console.log(p);
	}
	
	foo() // 100
	
	x = 100;
	foo() // 101

上面的代码表明每次调用`foo()`时都会对参数`p`进行求值。

在设置函数参数默认值时，最好把需要设置的参数位置放在末尾，如下面的代码：

	// 例一
	function f(x = 1, y) {
	  return [x, y];
	}
	
	f() // [1, undefined]
	f(2) // [2, undefined])
	f(, 1) // 报错
	f(undefined, 1) // [1, 1]
	
	// 例二	
	function f(x, y = 5, z) {
	  return [x, y, z];
	}
	
	f() // [undefined, 5, undefined]
	f(1) // [1, 5, undefined]
	f(1, ,2) // 报错
	f(1, undefined, 2) // [1, 5, 2]

如果有多个参数需要设置默认值，最好封装成一个对象，并将默认值写在该对象中而不是在函数默认值中，如下面的代码：

	function fetch(url, { body = '', method = 'GET', headers = {} }) {
	  console.log(method);
	}
	
	fetch('http://example.com', {})
	// "GET" 这样写参数默认值需要显示地给出第二个参数为{}空对象
	
	fetch('http://example.com')
	// 报错

	// 应该这样写

	function fetch(url, { body = '', method = 'GET', headers = {} } = {}) {
	  console.log(method);
	}
	
	fetch('http://example.com')
	// "GET"

### 函数对象的length属性 ###

`length`的含义为函数预期传入的参数个数，如果有参数指定了默认值，`length`的值会减去，并且`...rest`参数也不计入在`length`。还有一点，如果设置的默认值参数不是尾参数，那么包括该默认值以后的参数都不计入在`length`中。事实上可以用`arguments.length`直接得到传入的参数总个数。总结，是个SB属性，感觉没啥用？

### 函数作用域 ###

在设置参数默认值并进行声明初始化时，参数会形成一个单独作用域，初始化结束后这个作用域就会消失，只有在设置了参数默认值时才会有这种行为。

注：只有在函数调用初始化时才会有这种行为。

例：

	var x = 1;
	function f(x, y = x) {
	// 注意，这里y = x中的x是函数参数的第一个参数x
	  console.log(y);
	}
	f(2) // 2

例：

	let x = 1;
	function f(y = x) {
	  let x = 2;
	  console.log(y);
	}
	f() // 1

例：

	function f(y = x) {
	  let x = 2;
	  console.log(y);
	}
	f() // ReferenceError: x is not defined

例子表明函数体内部的变量与函数参数之间没有联系，函数在调用时并不能读取到内部的变量。如果在调用时未传入参数，且参数的默认值未定，则会报错。这还是一个函数作用域链的问题，在调用时参数只能访问到默认参数和函数外部变量，不能读到函数内部定义的变量，

当参数默认值是一个函数时，也遵循这个规则

	let foo = 'outer';
	function bar(func = () => foo) {
	  let foo = 'inner';
	  console.log(func());
	}
	bar(); // outer

函数参数`func`的默认值是一个匿名函数并且返回的是变量`foo`，此时`foo`指向全局变量`foo = 'outer'`。

一个更复杂的例子

	var x = 1;
	function foo(x, y = function() { x = 2; }) {
	// 参数y的默认值是一个匿名函数，匿名函数中的x是第一个参数x
	  var x = 3;
	// 这个x是函数foo内部声明的x，与全局变量x无关，与第一个参数无关
	  y();
	// 这里调用了y，事实上也只改变了foo函数的第一个参数x，但此时在函数内部已经无法访问到第一个参数x了，只能访问到内部变量x和外部变量window.x
	  console.log(x);
	}
	foo() // 3
	x // 1


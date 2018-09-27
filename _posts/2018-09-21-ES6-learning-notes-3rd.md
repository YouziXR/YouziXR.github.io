# ES6新特性学习第三篇 #

## 前言 ##

承接上一篇，本篇从数组扩展开始学习。

## 第一篇前言 ##

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 原著:阮一峰 ") 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ "阮一峰ES6")

## 数组的扩展 ##

### 扩展运算符 ###

形如spread = `...`，是rest参数的一种逆运算，拆分一个数组转换成用逗号分隔的参数序列。

用作函数参数

	function push(array, ...items) {
	  array.push(...items);
	}
	function add(x, y) {
	  return x + y;
	}
	const numbers = [4, 38];
	add(...numbers) // 42


与正常的函数参数结合

	function f(v, w, x, y, z) { }
	const args = [0, 1];
	f(-1, ...args, 2, ...[3]);

扩展运算符后可以放置表达式

	const arr = [
	  ...(x > 0 ? ['a'] : []),
	  'b',
	];

扩展运算符后是一个空数组，则不产生效果

	[...[], 1]
	// [1]

#### 替代函数apply方法 ####

一般的apply方法是将数组作为函数参数传给需要执行的函数

	// ES5 的写法
	function f(x, y, z) {
	  // ...
	}
	var args = [0, 1, 2];
	f.apply(null, args);
	
	// ES6的写法
	function f(x, y, z) {
	  // ...
	}
	let args = [0, 1, 2];
	f(...args);

一个求数组最大元素的写法

	// ES5 的写法
	Math.max.apply(null, [14, 3, 77])
	// ES6 的写法
	Math.max(...[14, 3, 77])
	// 等同于
	Math.max(14, 3, 77);

由于`Math.max`函数只能接受一组数作为参数，所以要在数组上用这个方法需把数组转换成参数序列。

另一个`push`函数的例子。

	// ES5的 写法
	var arr1 = [0, 1, 2];
	var arr2 = [3, 4, 5];
	Array.prototype.push.apply(arr1, arr2);
	// ES6 的写法
	let arr1 = [0, 1, 2];
	let arr2 = [3, 4, 5];
	arr1.push(...arr2);

#### 应用 ####

复制数组，直接复制是浅拷贝，拷贝的是数组指向堆栈的指针，不会克隆一个新的数组。

	const a1 = [1, 2];
	const a2 = a1;
	a2[0] = 2;
	a1 // [2, 2]
	ES5复制数组的方法。
	const a1 = [1, 2];
	const a2 = a1.concat();
	a2[0] = 2;
	a1 // [1, 2]

扩展运算符写法。

	const a1 = [1, 2];
	// 写法一
	const a2 = [...a1];
	// 写法二
	const [...a2] = a1;

合并数组

	const arr1 = ['a', 'b'];
	const arr2 = ['c'];
	const arr3 = ['d', 'e'];
	// ES5 的合并数组
	arr1.concat(arr2, arr3);
	// [ 'a', 'b', 'c', 'd', 'e' ]
	// ES6 的合并数组
	[...arr1, ...arr2, ...arr3]
	// [ 'a', 'b', 'c', 'd', 'e' ]

注意这两种方法都是浅拷贝，是对原数组成员的引用。

与解构赋值结合，生成数组。

	// ES5
	a = list[0], rest = list.slice(1)
	// ES6
	[a, ...rest] = list
	// 例子
	const [first, ...rest] = [1, 2, 3, 4, 5];
	first // 1
	rest  // [2, 3, 4, 5]
	const [first, ...rest] = [];
	first // undefined
	rest  // []
	const [first, ...rest] = ["foo"];
	first  // "foo"
	rest   // []
	// 扩展运算符要放在最后一位
	const [...butLast, last] = [1, 2, 3, 4, 5];
	// 报错
	const [first, ...middle, last] = [1, 2, 3, 4, 5];
	// 报错
	
字符串，扩展运算符可以将字符串转化成数组。
	
	[...'hello']
	// [ "h", "e", "l", "l", "o" ]
	'x\uD83D\uDE80y'.length // 4
	[...'x\uD83D\uDE80y'].length // 3
	
	let str = 'x\uD83D\uDE80y';
	str.split('').reverse().join('')
	// 'y\uDE80\uD83Dx'
	[...str].reverse().join('')
	// 'y\uD83D\uDE80x'

对于实现了Iterator接口的对象，可以用扩展运算符转化成数组。如Map，Set，Generator函数。

	let nodeList = document.querySelectorAll('div');
	let array = [...nodeList];

返回一个`nodeList`对象，是一个类数组对象，因为这个对象有Iterator接口，所以可以用扩展运算符，也可以用`Array.from`方法。

	let arrayLike = {
	  '0': 'a',
	  '1': 'b',
	  '2': 'c',
	  length: 3
	};
	// TypeError: Cannot spread non-iterable object.
	let arr = [...arrayLike];

`arrayLike`是一个普通的对象，但包含了一个`length`属性，没有Iterator接口，所以不能用扩展运算符，只能用`Array.from`方法。

### Array.from() ###

该方法将两类对象转化成数组，类数组(array-like)对象和可遍历(iterable)对象。可遍历对象包括新增的Set和Map。

	let arrayLike = {
	    '0': 'a',
	    '1': 'b',
	    '2': 'c',
	    length: 3
	// 注意一定要加上length属性，否则转化后的数组长度是0
	};
	// ES5的写法
	var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']
	// ES6的写法
	let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']

比较常见的类数组对象有DOM操作返回的NodeList集合，函数内部参数`arguments`对象。

同样地，只要部署了Iterator接口的数据结构都可以用`Array.from`将其转成数组。

	Array.from('hello')
	// ['h', 'e', 'l', 'l', 'o']
	let namesSet = new Set(['a', 'b'])
	Array.from(namesSet) // ['a', 'b']

如果参数是一个真正的数组，调用`Array.from`会返回一个相同的新数组。

	Array.from([1, 2, 3])
	// [1, 2, 3]

和上一节的扩展运算符不一样的是，该方法可以把类数组对象也转换成数组，但要求对象有`length`属性。事实上只要对象有`length`属性，都可以通过`Array.from`方法把这个对象转化成数组。

	console.log(Array.from({
		'1': 'ayou',
		'4': 9,
		'10': 10,
		length: 10
	})); // 除了index下标是1,4之外的其他数组元素全是undefined

对于未部署该方法的环境，可以用ES5的`slice`方法代替。

`Array.from`方法还可以接受第二个参数，作用类似于数组的`map`方法，用来对每个数组元素进行操作，将处理后的值放进返回的数组。
	
	Array.from(aryLike, x => x + 1);
	Array.from(aryLike).map( x => x + 1);
	// 效果一样
	// DOM节点的例子
	let spans = document.querySelectorAll('span.name');
	// map()
	let names1 = Array.prototype.map.call(spans, s => s.textContent);
	// Array.from()
	let names2 = Array.from(spans, s => s.textContent)

将数组中布尔值是`false`的成员转化成`0`

	Array.from([1, , 2, , 3], (n) => n || 0)
	// [1, 0, 2, 0, 3]
	// 返回数据类型的例子
	function typesOf () {
	  return Array.from(arguments, value => typeof value)
	}
	typesOf(null, [], NaN)
	// ['object', 'object', 'number']

如果`map`函数里还用到了`this`，还可以传入`Array.from`第三个参数，用来绑定`this`。

这个方法可以把各种值转化成数组，并且提供了`map`功能，这就意味着只要有一个原始的数据结构，就可以对其中的值进行处理然后转化成规范的数据结构，然后就能使用很多数组方法了。

	Array.from({ length: 2}, () => 'jack');
	// ['jack', 'jack']

方法中的第一个参数指定了第二个参数的运行的次数，这种特性可以让该方法很灵活。

方法的另一个应用就是将字符串转化成数组，返回字符串的长度，因为能正确处理Unicode字符，避免了JS将大于`\uFFFF`的字符算成两个字符的bug。

	function countSymbols(string) {
	  return Array.from(string).length;
	}

### Array.of() ###

该方法将一组值转化为数组，主要目的是为了弥补数组构造函数`Array()`的不足，在使用构造函数构造数组时，参数个数不同会导致构造函数行为差异。

	Array.of(3, 11, 8) // [3,11,8]
	Array.of(3) // [3]
	Array.of(3).length // 1
	Array() // []
	Array(3) // [, , ,]
	Array(3, 11, 8) // [3, 11, 8]
	Array.of() // []
	Array.of(undefined) // [undefined]
	Array.of(1) // [1]
	Array.of(1, 2) // [1, 2]

这个方法基本上能代替`Array()、new Array()`用来构造数组，且不会因为参数不同导致重载。显然这个方法总返回参数值组成的数组，没有参数就返回一个空数组。

### 数组的copyWithin() ###

这个方法可以把指定位置的元素复制到其他位置，会覆盖掉原有的元素，这个方法会修改当前数组。

方法接受三个参数。

`arr.copyWithin(target[, start[, end]])`，`target（必需）`从这个index开始覆盖数据，参数为负值则+length；`start（可选）`从这个index开始读取数据，默认0，为负值+length；`end（可选）`从这个index前停止数据，不包括end，默认是数组长度，即从start开始到最后一个元素，为负值+length。

三个参数应该都是数值，如果不是会自动转为数值。

例子：

	[1, 2, 3, 4, 5].copyWithin(-2);
	// [1, 2, 3, 1, 2]
	[1, 2, 3, 4, 5].copyWithin(0, 3);
	// [4, 5, 3, 4, 5]
	[1, 2, 3, 4, 5].copyWithin(0, 3, 4);
	// [4, 2, 3, 4, 5]
	[1, 2, 3, 4, 5].copyWithin(-2, -3, -1);
	// [1, 2, 3, 3, 4]

从start开始复制到end前一个元素，然后从target开始覆盖已经复制的值。

### find() findIndex() ###

数组实例的`find()`方法用于找出第一个符合条件的数组成员，参数是一个回调函数，所有数组元素依次执行回调函数，直到找出第一个返回值为`true`的元素，然后返回这个元素，没有符合条件的就返回`undefined`。

	[].find(function(value, index, array){
		// doSomething
	})

`find`函数的回调函数可以接受三个参数，依次为当前的值、当前的位置和原数组。

`findIndex`方法返回第一个符合条件的数组元素的位置，如果都不符合则返回`-1`。

	[1, 5, 10, 15].findIndex(function(value, index, arr) {
	  return value > 9;
	}) // 2

这两个方法都可以接受第二个参数，用来绑定回调函数的`this`。

	function f(v){
	  return v > this.age;
	}
	let person = {name: 'John', age: 20};
	[10, 12, 26, 15].find(f, person);    // 26

上面的代码里，`find`函数接受了第二参数`person`，指定了回调函数的`this`对象指向了`person`。

另外，这两个方法都可以发现`NaN`，弥补了数组的`indexOf`方法的不足。

	[NaN].indexOf(NaN)
	// -1
	[NaN].findIndex(y => Object.is(NaN, y))
	// 0

`indexOf`无法识别，但`findIndex`可以借助`Object.is`方法做到。

### fill() ###

该方法使用给定值填充数组。用于对空数组的初始化很方便，数组中原有的元素会被覆盖。

`array.fill(value, start, end)`参数有3个，`value（必需）`是用来填充元素的值，`start（可选）`起始index，默认为0，`end`终止前index，默认为length。参数为负时自动加上length，返回修改后的数组。

需要注意的是，当一个对象被传递给`fill`方法时，填充的数组是这个对象的引用，浅拷贝，复制的是地址，而不是实际值。

	let arr = new Array(3).fill({name: "Mike"});
	arr[0].name = "Ben";
	arr
	// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]
	let arr = new Array(3).fill([]);
	arr[0].push(5);
	arr
	// [[5], [5], [5]]

### entries() keys() values() ###

三个方法都用于遍历数组，是数组实例的方法，返回一个遍历器对象，可以用`for...of`循环遍历，区别是`keys()`对键名遍历，`values()`对键值遍历，`entries`对键值对遍历。

如果不用`for...of`循环，可以手动调用遍历器对象的`next`方法进行遍历。

	for (let index of ['a', 'b'].keys()) {
	  console.log(index);
	}
	// 0
	// 1
	for (let elem of ['a', 'b'].values()) {
	  console.log(elem);
	}
	// 'a'
	// 'b'
	for (let [index, elem] of ['a', 'b'].entries()) {
	  console.log(index, elem);
	}
	// 0 "a"
	// 1 "b"

	let letter = ['a', 'b', 'c'];
	let entries = letter.entries();
	console.log(entries.next().value); // [0, 'a']
	console.log(entries.next().value); // [1, 'b']
	console.log(entries.next().value); // [2, 'c']

### includes() ###

该方法返回一个布尔值，是数组实例方法，表示某个数组是否包含给定值。

`array.includes(searchElement, fromIndex)`，参数`searchElement（必需）`需要查找的元素，`fromIndex（可选）`从该索引处开始查找。默认为0。


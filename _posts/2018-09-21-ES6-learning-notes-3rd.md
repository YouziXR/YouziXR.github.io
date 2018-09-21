# ES6新特性学习第三篇 #

## 前言 ##

承接上一篇，本篇从数组扩展开始学习。

## 第一篇前言 ##

本文建立在学习阮一峰老师的ES6教程之上，总结了一些我自己认为重要的点，主要面向我本人，偏向于学习笔记的形式，主要参考 [http://jsrun.net/tutorial/cZKKp](http://jsrun.net/tutorial/cZKKp "ES6 全套教程 ECMAScript6 原著:阮一峰 ") 和 [http://es6.ruanyifeng.com/](http://es6.ruanyifeng.com/ "阮一峰ES6")

## 数组的扩展 ##

### 扩展运算符 ###

形如spread = `...`，是rest参数的一种逆运算，拆分一个数组转换成用逗号分隔的参数序列。

用作函数参数，

	function push(array, ...items) {
	  array.push(...items);
	}
	function add(x, y) {
	  return x + y;
	}
	const numbers = [4, 38];
	add(...numbers) // 42


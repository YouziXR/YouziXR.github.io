---
layout: post
title: '面试总结'
date: 2020-9-16 12:05:29
author: 'Youzi'
catalog: true
tags:
  - JavaScript
  - 面试
---

# 面经

## VueRouter 相关

- 原理
- 两种模式的优劣
- 如何实现页面部分渲染

## Vue3.0 原理

根据回答方向，确定后续问题（compositionAPI、响应式的原理改变等等，待补充）

### 2.0 和 3.0 的区别

1. 重构响应式系统，使用 Proxy 替换 Object.defineProperty，使用 Proxy 优势： •可直接监听数组类型的数据变化 •监听的目标为对象本身，不需要像 Object.defineProperty 一样遍历每个属性，有一定的性能提升 •可拦截 apply、ownKeys、has 等 13 种方法，而 Object.defineProperty 不行 •直接实现对象属性的新增/删除
2. 新增 Composition API，更好的逻辑复用和代码组织
3. 重构 Virtual DOM •模板编译时的优化，将一些静态节点编译成常量 •slot 优化，将 slot 编译为 lazy 函数，将 slot 的渲染的决定权交给子组件 •模板中内联事件的提取并重用（原本每次渲染都重新生成内联函数）
4. 代码结构调整，更便于 Tree shaking，使得体积更小
5. 使用 Typescript 替换 Flow

## 网络原理

- 在浏览器中从 URL 输入到页面展现到底发生什么？
- 浏览器渲染过程
- 渲染中读到`<script> / <style>`会发生什么，`script`标签的阻塞情况，如何避免阻塞

## 事件循环

- 宏任务、微任务，同步任务的执行顺序（同步任务执行完后会 check 异步任务队列）
- requestAnimationFrame 相关

## 防抖节流

## 安全、攻击 XSS，CSRF

## HTTPS / HTTP1.X-2.0-3.0

## new 做了什么

new 运算接受一个构造器和一组调用参数，实际上做了几件事：

- 以构造器的 prototype 属性（注意与私有字段`[[prototype]]`的区分）为原型，创建新对象；
- 将 this 和调用参数传给构造器，执行；
- 如果构造器返回的是对象，则返回，否则返回第一步创建的对象。如果构造函数内部返回了一个对象字面量，那这里返回的就应该是构造函数内部的对象，而不是第一步创建的对象。

new 这样的行为，试图让函数对象在语法上跟类变得相似，但是，它客观上提供了两种方式，一是在构造器中添加属性，二是在构造器的 prototype 属性上添加属性。

## 华为 OD 笔试

1. 全排列问题，具体就是给一个`n, k`，从小到大排序`1~n`的全排列，输出第 k 个值。好棒，第一题我就不会，慌了一下然后跳过了。

2. 字符串处理。`M, G, T`三个单位的转换，第一行是有多少组字符串，给定一组字符串，形如`20G1M3T, 1T2M3G`这样的，比较大小，进制就是 1024，输出从小到大排序后的字符串数组。

**思路：写一个处理单个字符串的函数，转换成整型数然后比较大小；**

> 备注：这个代码通过率95%，当时也是有时间限制所以没有接着debug了

```javascript
let n = Number(readline());
let l = [];
function mgt(str) {
  let sarr = str.split('');
  let sum = 0;
  let tmpS = '';
  for (let el of sarr) {
    /* if (el !== 'M' || el !== 'G' || el !== 'T') {
      tmpS += el;
    } else { */
    switch (el) {
      case 'M':
        sum += Number(tmpS);
        tmpS = '';
        break;
      case 'G':
        sum = sum + 1024 * Number(tmpS);
        tmpS = '';
        break;
      case 'T':
        sum = sum + 1024 * 1024 * Number(tmpS);
        tmpS = '';
        break;
      default:
        tmpS += el;
        break;
      // }
    }
  }
  return sum;
}
while (n--) {
  l.push(readline());
}
l.sort((a, b) => {
  return mgt(a) - mgt(b);
});
for (const i of l) {
  print(i);
}
```

3. 贪心问题。设定场景：硬盘容量 500，输入一串数字字符串`1,100,200,500`表示每个文件的文件大小，问最少需要多少个硬盘才能装下全部文件，文件不能进行拆分之类的操作。

**思路：一开始没想到贪心算法，想着遍历去凑几个文件加起来到 500，后来思考了下，如果把这些文件大小排个序，从小到大，那么我们首先对一个文件大小 x，要找的就是 500-x，如果没找到，就接着用贪心的思想，把前面都没找到的配对的文件组合起来，看看是不是凑到了 500；我也不太会证明这个算法；**

> 备注：这个代码通过率100%了。

```javascript
let s = readline();
// s = '1,200,100,300';
let arr = s.split(',');
let map = new Map();
arr = arr.map(el => {
  let t = Number(el);
  let i = map.get(t);
  if (i) {
    map.set(t, ++i);
  } else {
    map.set(t, 1);
  }
  return Number(el);
});
arr.sort((a, b) => a - b);
let sum = 0;
let tmpSum = 0;
for (let i = 0; i < arr.length; i++) {
  const a = arr[i];
  let tmpA = map.get(a);
  if (tmpA === 0) {
    // 为0的情况，表示这个数已经被用过了，接着往下就行
    continue;
  }
  if (a === 500) {
    // 等于500就直接继续往下
    sum++;
    map.set(a, --tmpA);
    continue;
  }
  // 标记一下这个值被用过一次了，给它-1
  map.set(a, --tmpA);
  let tmp = map.get(500 - a);
  if (tmp) {
    // 找到了
    sum++;
    map.set(500 - a, --tmp);
    continue;
  }
  // 没找到
  if (tmpSum + a === 500) {
    // 如果前面没配对成功的数字总和，和现在的数字加起来正好是500，就正好也是找到了的情况
    sum++;
    tmpSum = 0;
    // continue;
  } else if (tmpSum + a > 500) {
    // 如果大于500，那表示前面没配对成功的数字总和也不可能再配对了
    // 那就让前面的数字单独占一个硬盘，且让当前数字总和等于现在的这个数字
    sum++;
    tmpSum = a;
    // continue;
  } else {
    // 如果小于500，就接着贪心下一个
    tmpSum += a;
  }
  /* for (let j = i + 1; j < arr.length; j++) {
    const b = arr[j];
  } */
}
if (tmpSum !== 0) {
  // 还需要看最后一个数字是不是被分配了，
  // 如果前面的数字总和是0了,那就表示全部分配完了
  // 如果不是0,表示还要给这些数字再分配一个硬盘
  sum++;
}
print(sum);
// function findTmpSum(n) {}
```

title: 从函数式编程的特点出发，理解函数式编程
date: 2017/8/8
---

# 从函数式编程的特点出发，理解函数式编程

## 前言

什么是函数式编程？这个问题可能很难组织语言描述清楚。很多资料对函数式编程的定义让人难以琢磨，使得大家对函数式编程望而却步。本文尝试从函数式编程的特点出发，阐述函数式编程的优点，希望能启发大家对函数式编程的理解。


## 递归

如果一种编程语言里没有循环，如何处理重复的操作？函数式编程的一个重要特点就是没有循环，在一些纯函数式编程语言（如haskell）里更是没有循环的语法。
在函数式编程里，所有的重复操作通过递归来实现。这听起来可能很不可思议，但是如果你尝试从这一角度编写代码，你会发现一个新的大陆。


我们从几个小例子，体验一些递归编程的思路。


```js
// 阶乘
function factorial (n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

// 求数组最大值
function maximum (list) {
  if (list.length === 0) throw "maximum of empty list";
  if (list.length === 1) return list[0];
  
  const [x, ...xs] = list;
  const maxTail = maximum(xs);
  return x > maxTail ? x : maxTail;
}

// 数组排序
function quicksort(list) {
  if (list.length === 0) return [];
  
  const [x, ...xs] = list;
  const smaller = quicksort(xs.filter(d => d <= x));
  const bigger = quicksort(xs.filter(d => d > x));
  return [...smaller, x, ...bigger];
}

// reverse
function reverse(list) {
  if (!list.length) return [];
  const [x, ...xs] = list;
  return [...reverse(xs), x];
}
```


以上的范例，是不是与大家使用循环的思路相比，显得清新脱俗，更有bigger？这便是函数式编程的魅力。让我们来捋一捋其中的思路：


1. 判断一些‘边界条件’的结果（如以上1的阶乘，空数组，单元素数组);
2. 从一堆元素中取一个并做点事情后，假设这个函数已经能完成对应的功能，把余下的操作重新交给这个函数（递归）;
3. 对比、处理递归函数的返回值，返回正确的结果。


对比使用循环解决这些问题，我们可以发现函数式编程有以下特点：

*  用声明函数是什么的形式来写程序。不是像命令式语言那样命令电脑「要做什么」，而是通过用函数来描述出问题「是什么」，如「阶乘是指从1到某个数的乘积」，「一个串列中数字的和」是指把第一个数字跟剩余数字的和相加。

* 【没有变量】或者说变量一经定义就不会修改。这使得函数即不依赖外部的状态也不修改外部的状态，函数调用的结果不依赖调用的时间和位置（若以同样的参数调用同一个函数两次，得到的结果一定是相同），这样写的代码容易进行推理，不容易出错。这使得单元测试和调试都更容易。


## 高阶函数


高级函数，是指接受函数作为参数或返回函数作为结果的函数。由于javascript有函数是‘一等公民’、闭包等特性，可以方便的实现高阶函数。
下面有一些高阶函数的例子

```js
// map
function map(fn, list){
  if (!list.length) return [];
  const [x, ...xs] = list;
  return [fn(x), ...map(fn, xs)];
}

// filter
function filter(fn, list){
  if (!list.length) return [];
  const [x, ...xs] = list;
  const filterOther = filter(fn, xs);
  return fn(x) ? [x, ...filterOther] : filterOther;
}

// reduce
function reduce(fn, list, acc){
  if (!list.length) return acc;

  const [x, ...xs] = list;

  if (arguments.length <= 2) return reduce(fn, xs, x);
  return reduce(fn, xs, fn(acc, x));
}
```
**柯里化（curry）** 和 **代码组合（compose)** 是函数式编程里常用的两个概念。
在javascript中，curry 和 compose 的实现函数，也是高阶函数：

```js
// 柯里化, 把接受多个参数的函数变换成接受一个单一参数，并且返回接受余下的参数而且返回结果的新函数
function curry(fn) {
  return x => ((...arg) => fn(x ,...arg));
}

const add = (x, y) => x + y;
const curriedAdd = curry(add); // 柯里化 add 函数
curriedAdd(1)(2); // 3

// 代码组合
function compose(f, g)  {
  return (x) =>  f(g(x));
}

const square = x => x * x;
const multipliedByPi = x => x * Math.PI;
const areaOfCircle = compose(multipliedByPi, square); // compose 
```

有了以上两个方法，我们便可以组合出各种不同的函数：

```js
const first = list => list[0];
// 返回数组最后一个元素
const last = compose(first, reverse);
// 返回数组 >5 的元素集合
const gt5 = curry(filter)(x => x > 5);
// 数组求和
const sum = curry(reduce)(add);
// 返回数组的平方和
const squareSum = compose(sum, curry(map)(square));

// 在js中，我们可以使用另一种组合方式
const squareSum = list => list.map(square).reduce(add);
```


当然，你也可以自己编写高阶函数。许多js工具库(如lodash)也提供了许多有用的高阶函数。运用这些高阶函数，可以将方法组合起来，形成一个‘管道’。这意味着你不用再去根据过程实现新的方法。通过组合已有和必要的新函数，可以更可靠、快速的实现一个新方法，使代码简单而富有可读性。这也是函数式编程所倡导的‘声明函数’，通过定义问题"是什么"来解决问题。

## 总结


函数式编程强调程序执行的结果而非执行的过程，倡导利用若干简单的执行单元让计算结果不断渐进，逐层推导复杂的运算。它的优点有：引用透明，没有副作用；更接近人的语言，容易被理解。缺点是牺牲了一定的执行效率。


## Reference


* [HASKELL 趣學指南](https://www.gitbook.com/book/mno2/learnyouahaskell-zh/details)
* [[知乎] 什么是函数式编程思维？](https://www.zhihu.com/question/28292740)
* [[wikipedia] 函数编程语言](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)

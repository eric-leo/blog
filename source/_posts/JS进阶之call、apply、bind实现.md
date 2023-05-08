---
title: JS进阶之call、apply、bind实现
date: 2018-07-19 21:19:32
type: "js-base"
tag: js
description:
keywords:
top_img:
mathjax:
katex:
aside:
---


### 引言
在实际开发过程中，对于函数封装时，不确定外部是谁调用的，调用函数内部方法时，有可能是window调用这时就会报错，常使用call、apply、bind来绑定this指向。

### Function.prototype.call()
> 定义：call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

> 注意：该方法和apply()类似，区别在于，call()可以接收若干参数，而apply（）接收的是一个包含多个参数的数组。

> 语法：function.call(thisArg, arg1, arg2, ...)

#### 用途:

call方法实现继承:
``` js
function Product(name, price) {
  this.name = name;
  this.price = price;
}

function Food(name, price) {
  Product.call(this, name, price);
  this.category = 'food';
}

console.log(new Food('cheese', 5).name);
// expected output: "cheese"

```

call方法调用匿名函数：
```js
var animals = [
    { species: 'Lion', name: 'King' },
    { species: 'Whale', name: 'Fail' }
  ];
  
  for (var i = 0; i < animals.length; i++) {
    (function(i) {
        console.log('#' + i + ' ' + this.species + ': ' + this.name) }
    ).call(animals[i], i);
  }
```

call方法指定上下文的this:
```js
function greet() {
  var reply = [this.animal, 'typically sleep between', this.sleepDuration].join(' ');
  console.log(reply);
}
var obj = {
  animal: 'cats', sleepDuration: '12 and 16 hours'
};
greet.call(obj);
// cats typically sleep between 12 and 16 hours
```
#### 手写call实现原理：

``` js
/** 
 * call核心思路：
  将函数设为对象的属性
  执行&删除这个函数
  指定this到函数并传入给定参数执行函数
  如果不传入参数，默认指向为 window
  */
Function.prototype.call2 = function(content = window) {
  content.fn = this;
  const args = [...arguments].slice(1);
  const res = content.fn(...args);
  delete content.fn;
  return res;
}
```

### Function.prototype.apply()
> 定义：apply() 方法调用一个具有给定this值的函数，以及作为一个数组（或类似数组对象）提供的参数。

> 注意：call()方法的作用和 apply() 方法类似，区别就是call()方法接受的是参数列表，而apply()方法接受的是一个参数数组。

> 语法：func.apply(thisArg, [argsArray])

#### 用途:

apply 将数组添加到另一个数组：

```js
var array = ['a', 'b'];
var elements = [0, 1, 2];
array.push.apply(array, elements);
console.log(array); // ["a", "b", 0, 1, 2]
```

apply 找出最大值和最小值
```js
var numbers = [5, 6, 2, 3, 7];
var max = Math.max.apply(null, numbers)
var min = Math.min.apply(null, numbers);
```

#### 手写apply实现原理：

```js
Function.prototype.apply2 = function(content = window) {
  content.fn = this;
  let res;
  let args = [...arguments][1]
  if (args) {
    res = content.fn(args);
  } else {
    res = content.fn();
  }
  delete content.fn;
  return res;
}
```

### Function.prototype.bind()
> 定义：bind() 方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

> 注意：该方法不会像call和apply一样立即调用绑定的函数，需要手动执行

> 语法：function.bind(thisArg[, arg1[, arg2[, ...]]])

#### 用途:

创建绑定函数：
```js
this.x = 9;    // 在浏览器中，this 指向全局的 "window" 对象
var module = {
  x: 81,
  getX: function() { return this.x; }
};

module.getX(); // 81

var retrieveX = module.getX;
retrieveX();   
// 返回 9 - 因为函数是在全局作用域中调用的

// 创建一个新函数，把 'this' 绑定到 module 对象
// 新手可能会将全局变量 x 与 module 的属性 x 混淆
var boundGetX = retrieveX.bind(module);
boundGetX(); // 81

```

偏函数：
```js
function list() {
  return Array.prototype.slice.call(arguments);
}

function addArguments(arg1, arg2) {
    return arg1 + arg2
}

var list1 = list(1, 2, 3); // [1, 2, 3]

var result1 = addArguments(1, 2); // 3

// 创建一个函数，它拥有预设参数列表。
var leadingThirtysevenList = list.bind(null, 37);

// 创建一个函数，它拥有预设的第一个参数
var addThirtySeven = addArguments.bind(null, 37); 

var list2 = leadingThirtysevenList(); 
// [37]

var list3 = leadingThirtysevenList(1, 2, 3); 
// [37, 1, 2, 3]

var result2 = addThirtySeven(5); 
// 37 + 5 = 42 

var result3 = addThirtySeven(5, 10);
// 37 + 5 = 42 ，第二个参数被忽略
```

#### 手写bind实现原理
```js
Function.prototype.bind = function() {
    var thatFunc = this, thatArg = arguments[0];
    var slice = Array.prototype.slice;
    var args = slice.call(arguments, 1);
    if (typeof thatFunc !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - ' +
             'what is trying to be bound is not callable');
    }
    return function(){
      var funcArgs = args.concat(slice.call(arguments))
      return thatFunc.apply(thatArg, funcArgs);
    };
};
```


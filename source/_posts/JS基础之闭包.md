---
title: JS基础之闭包
date: 2018-05-12 09:31:29
type: "js-base"
tag: js
description:
keywords:
top_img:
mathjax:
katex:
aside:
---

#### 什么是闭包？
> 红宝石书对于闭包的解释：闭包是指有权访问另外一个函数作用域中的变量的函数

> MDN对于闭包的解释：闭包是函数和声明该函数的词法环境的组合。

#### 代码理解

```javascript
function A() {
  let a = 1
  function B() {
      console.log(a)
  }
  return B
}
```

从上面的例子中可以看到：函数 A 返回了一个函数 B，并且函数 B 中使用了函数 A 的变量，所以函数 B 就被称为闭包。

#### 闭包的三个特性

##### 1.闭包可以访问当前函数以外的变量

```javascript
function getOuter(){
  var date = '512';
  function getDate(str){
    console.log(str + date);  //访问外部的date
  }
  return getDate('今天是：'); //"今天是：512"
}
getOuter();
```

##### 2.即使外部函数已经返回，闭包仍能访问外部函数定义的变量

```javascript
function getOuter(){
  var date = '512';
  function getDate(str){
    console.log(str + date);  //访问外部的date
  }
  return getDate;     //外部函数返回
}
var today = getOuter();
today('今天是：');   //"今天是：512"
today('明天不是：');   //"明天不是：512"
```

##### 3.闭包可以更新外部变量的值

```javascript
function updateCount(){
  var count = 0;
  function getCount(val){
    count = val;
    console.log(count);
  }
  return getCount;     //外部函数返回
}
var count = updateCount();
count(512); //512
count(513); //513
```

#### 经典面试题

**循环中使用闭包解决 var 定义函数的问题**

```javascript
for ( var i=1; i<=5; i++) {
	setTimeout( function timer() {
		console.log( i );
	}, i*1000 );
}
```

首先因为 setTimeout 是个异步函数，所有会先把循环全部执行完毕，这时候 i 就是 6 了，所以会输出一堆 6。

解决办法:
1.使用闭包
```javascript
for (var i = 1; i <= 5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j);
    }, j * 1000);
  })(i);
}
```

2.使用 setTimeout 的第三个参数
```javascript
for ( var i=1; i<=5; i++) {
	setTimeout( function timer(j) {
		console.log( j );
	}, i*1000, i);
}
```

3.使用 let 定义 i

```javascript
for ( let i=1; i<=5; i++) {
	setTimeout( function timer() {
		console.log( i );
	}, i*1000 );
}
```

因为对于 let 来说，他会创建一个块级作用域，相当于

```javascript
{ // 形成块级作用域
  let i = 0
  {
    let ii = i
    setTimeout( function timer() {
        console.log( ii );
    }, i*1000 );
  }
  i++
  {
    let ii = i
  }
  i++
  {
    let ii = i
  }
  ...
}
```

#### 闭包的缺陷

- 闭包的缺点就是常驻内存会增大内存使用量，并且使用不当很容易造成内存泄露。
- 如果不是因为某些特殊任务而需要闭包，在没有必要的情况下，在其它函数中创建函数是不明智的，因为闭包对脚本性能具有负面影响，包括处理速度和内存消耗。




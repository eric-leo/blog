---
title: JS进阶之EventLoop
date: 2019-01-14 15:33:54
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

开始介绍EventLoop之前我们先了解一些前置基础知识——线程和进程。

### 进程和线程

#### 进程

- CPU承担了所有的计算任务
- 进程是CPU资源分配的最小单位
- 在同一个时间内，单个CPU只能执行一个任务，只能运行一个进程
- 如果有一个进程正在执行，其它进程就得暂停
- CPU使用了时间片轮转的算法实现多进程的调度

#### 线程

- 线程是 CPU调度的最小单位
- 一个进程可以包括多个线程，这些线程共享这个进程的资源

### chrome浏览器进程

- 浏览器是多进程的
- 每一个TAB页就是一个进程
- 浏览器主进程
    - 控制其它子进程的创建和销毁
    - 浏览器界面显示，比如用户交互、前进、后退等操作
    - 将渲染的内容绘制到用户界面上
- 渲染进程就是我们说的浏览器内核
    - 负责页面的渲染、脚本执行、事件处理
    - 每个TAB页都有一个渲染进程
- 网络进程 处理网络请求、文件访问等操作
- GPU进程 用于3D绘制
- 第三方插件进程

### 渲染进程

- GUI渲染线程
    - 渲染、布局和绘制页面
    - 当页面需要重绘和回流时，此线程就会执行
    - 与JS引擎互斥

- JS引擎线程
    - 负责解析执行JS脚本
    - 只有一个JS引擎线程(单线程)
    - 与GUI渲染线程互斥

- 事件触发线程
    - 用来控制事件循环(鼠标点击、setTimeout、Ajax等)
    - 当事件满足触发条件时，把事件放入到JS引擎所有的执行队列中

- 定时器触发线程
    - setInterval和setTimeout所在线程
    - 定时任务并不是由JS引擎计时，而是由定时触发线程来计时的
    - 计时完毕后会通知事件触发线程

- 异步HTTP请求线程
    - 浏览器有一个单独的线程处理AJAX请求
    - 当请求完毕后，如果有回调函数，会通知事件触发线程

### EventLoop

在了解 Event Loop 之前，我们先来熟悉一下执行栈和事件队列这两个概念。

#### 执行栈

当我们调用一个方法的时候，js会生成一个与这个方法相对应的执行环境，也叫执行上下文，这个执行环境存在着这个方法的私有作用域、参数、this对象等等。因为js是单线程的，同一时间只能执行一个方法，所以当一系列的方法被依次调用的时候，js会先解析这些方法，把其中的同步任务按照执行顺序排队到一个地方，这个地方叫做执行栈。

#### 事件队列

当我们发出一个ajax请求，他并不会立刻返回结果，为了防止浏览器出现假死或者空白，主线程会把这个异步任务挂起(pending)，继续执行执行栈中的其他任务，等异步任务返回结果后，js会将这个异步任务按照执行顺序，加入到与执行栈不同的另一个队列，也就是事件队列。


![img1.jpg](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfcvu9kvj4j30gp0endg6.jpg)


如图所示：

- 主线程运行的时候会生成堆（heap）和栈（stack
- js从上到下解析方法，将其中的同步任务按照执行顺序排列到执行栈中
- 当程序调用外部的API时，比如ajax、setTimeout等，会将此类异步任务挂起，继续执行执行栈中的任务，等异步任务返回结果后，再按照执行顺序排列到事件队列中
- 主线程先将执行栈中的同步任务清空，然后检查事件队列中是否有任务，如果有，就将第一个事件对应的回调推到执行栈中执行，若在执行过程中遇到异步任务，则继续将这个异步任务排列到事件队列中
- 主线程每次将执行栈清空后，就去事件队列中检查是否有任务，如果有，就每次取出一个推到执行栈中执行，这个过程是循环往复的... 这个过程被称为“Event Loop 事件循环”。

以上的事件循环过程只是一个宏观的表述，实际上异步任务之间也不相同，执行优先级也有区别。不同的异步任务被分为两类：宏任务（macro task）和微任务（micro task）。我们将经常遇到的异步任务进行分类如下：

**宏任务**：setTimeout，setInterval，setImmediate，I/O(磁盘读写或网络通信)，UI交互事件

**微任务**：process.nextTick，Promise.then

![img.jpg](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfcvueisauj30g909x3yy.jpg)

前面我们介绍，事件循环会将其中的异步任务按照执行顺序排列到事件队列中。然而，根据异步事件的不同分类，这个事件实际上会被排列到对应的宏任务队列或者微任务队列当中去。

当执行栈中的任务清空，主线程会先检查微任务队列中是否有任务，如果有，就将微任务队列中的任务依次执行，直到微任务队列为空，之后再检查宏任务队列中是否有任务，如果有，则每次取出第一个宏任务加入到执行栈中，之后再清空执行栈，检查微任务，以此循环...

#### EventLoop练习题


为了加深印象，我们来几个例子🌰：


1.
```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');

```

输出：
script start
script end
promise1
promise2
setTimeout

2.
```javascript
async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2')
}
console.log('script start')
setTimeout(function () {
    console.log('setTimeout')
})
async1()
new Promise(function (resolve) {
    console.log('promise1')
    resolve()
}).then(function () {
    console.log('promise2')
})
console.log('script end')
```

输出：
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout

### Node中的EventLoop

- Node.js采用V8作为js的解析引擎，而I/O处理方面使用了自己设计的libuv
- libuv是一个基于事件驱动的跨平台抽象层，封装了不同操作系统一些底层特性，对外提供统一的API
- 事件循环机制也是它里面的实现
    - V8引擎解析JavaScript脚本并调用Node API
    - libuv库负责Node API的执行。它将不同的任务分配给不同的线程,形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎
    - V8引擎再将结果返回给用户

#### libuv

- 同步执行全局的脚本
- 执行所有的微任务，先执行nextTick中的所有的任务，再执行其它微任务
- 开始执行宏任务，共有6个阶段，从第1个阶段开始，会执行每一个阶段所有的宏任务


![img2.jpg](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfcwgztukpj30tf0d9dgy.jpg)

![img3.jpg](https://tva1.sinaimg.cn/large/007S8ZIlgy1gfcwgvnpt9j30xo0lxmzn.jpg)

#### setImmediate

setTimeout/setInterval取值范围是[1,2的32次方-1],超出范围初始化为1，所以 setTimeout(fn,0) = setTimeout(fn,1)

```javascript
setTimeout(function  () {
  console.log('timeout');
},0);
setImmediate(function  () {
  console.log('immediate');
});
```

```javascript
const fs = require('fs')
fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('timeout');
    }, 0)
    setImmediate(() => {
        console.log('immediate')
    })
})
```
#### process.nextTick 

nextTick独立于Event Loop,有自己的队列，每个阶段完成后如果存在nextTick队列会全部清空，优先级高于微任务

```javascript
setTimeout(() => {
    console.log('setTimeout1')
    Promise.resolve().then(function () {
        console.log('promise1')
    })
}, 0)
setTimeout(() => {
    console.log('setTimeout2')
    Promise.resolve().then(function () {
        console.log('promise2')
    })
}, 0)
setImmediate(() => {
    console.log('setImmediate1')
    Promise.resolve().then(function () {
        console.log('promise3')
    })
}, 0)

process.nextTick(() => {
    console.log('nextTick1');
    Promise.resolve().then(() => console.log('promise4'));
    process.nextTick(() => {
        console.log('nextTick2');
        Promise.resolve().then(() => console.log('promise5'));
        process.nextTick(() => {
            console.log('nextTick3')
            process.nextTick(() => {
                console.log('nextTick4')
            })
        })
    })
})
// nextTick1 nextTick2 nextTick3 nextTick4
//promise4 promise5 setTimeout1  promise1 setTimeout2 promise2  setImmediate1 promise3
```


---
title: JS进阶之节流、防抖实现
date: 2018-07-04 10:54:40
type: "js-base"
tag: js
description:
keywords:
top_img:
mathjax:
katex:
aside:
---

## 前言

对于**防抖**和**节流**，相信大家都已经听了太多太多，网上在这一块的资料也有很多。在真正使用它之前我也曾按照网上的教程手写过几个不同的版本，最近在工作中真正要用上的时候，才发现之前对其的理解还不够深刻。所以今天在这里好好的梳理一下它们，贴上一个比较全的实现方式，另外介绍一下实际工作中是怎样使用的。

## 为什么要用防抖节流？

首先想要说明的是为什么需要使用到**防抖**和**节流**，若是已经清楚的小伙可以直接跳过阅读后面的内容，若你和我一样在使用场景上还存在一些疑惑的话可以仔细看看这一块内容，我会尽量说的清晰一些。

**场景一**

有这么一个需求，用户在点击某个按钮的时候会触发一些任务事件。但若是他在很短的时间内连续的点击按钮，则会不停的触发任务事件，但我们想要这个任务事件在一定时间内只能触发一次(不管按钮被点击了几次)，然后过了这个时间再次点击就又可以触发了。

如图：

此时，像这种**频繁的触发任务事件，但却只想要它在一定时间内只执行一次，过了这个时间又能继续触发**的情况我们就称之为**防抖**。

但在实际工作中，我们发现上面避免按钮重复点击的情况一般都可以给按钮增加一个`loading`的效果来避免，用的更多的可能是下面的场景。

**场景二**

用户在输入框输入文字之后，需要将输入的文字发送给后台并获取到数据。我们知道这样的需求一般都是通过监听输入框内容的改变，然后将最新的内容传递给后台。但若是每输入一个字或者一个拼音就发送一个请求是否就会产生大量多余的请求，此时，使用**防抖**无疑是非常有必要的。

## 防抖

### 非立即执行防抖

相信大家看到上面的需求就知道，防抖的实现肯定离不开定时器了，所以我们先来实现一个简易版的防抖：

```javascript
// func是用户传入需要防抖的函数
// wait是等待时间
const debounce = (func, wait = 500) => {
  // 缓存一个定时器id
  let timer = 0
  // 这里返回的函数是每次用户实际调用的防抖函数
  return function(...args) { // 接收额外的参数
    if (timer) clearTimeout(timer) // 如果已经设定过定时器了就清空上一次的定时器
    timer = setTimeout(() => { // 开始一个新的定时器，延迟执行用户传入的方法
      func.apply(this, args)
    }, wait)
  }
}
```

可以看到，上面的`debounce`函数利用定时器将要执行的函数`func`放到事件队列中，等过了`wait`时长后就会执行。若是在还没过`wait`时长就再次触发了就会清空掉上一个定时器(也就是取消了上次的`func`)，从而重新计算`func`执行的时间。

了解了原理之后，我们就知道上面的函数并不会在一开始就执行，而是在过了`wait`时长之后才执行，像这种情况，我们称它为**非立即执行防抖**。

案例1🌰

```html
<body>
    <input id="text" onkeyup="changeInput(event)" />
    <p id="result"></p>
    <script>
        let text = document.querySelector('#text')
        let result = document.querySelector('#result')
        function changeInput (event) {
            // console.log(event.target.value)
            debouncePostValue(event.target.value)
        }
        function debounce (func, await = 500) {
            let timer = null;
            return function (...param) {
                if (timer) clearTimeout(timer)
                timer = setTimeout(() => {
                    func.apply(this, param)
                }, await)
            }
        }
        const debouncePostValue = debounce(postValue)
        function postValue (value) {
            result.innerHTML = value
        }
    </script>
</body>
```

### 立即执行防抖

**立即执行防抖**与上面的相似，只不过在第一次触发的时候会执行一次，然后要等待`await`时长之后才能再次触发。

```javascript
function debounce (func, await = 500) {
    let timer = null;
    return function (...param) {
        if (timer) clearTimeout(timer)
        let callNow = !timer
        timer = setTimeout(() => { // 在await之后将定时器清除
            timer = null
        }, await)
        if (callNow) func.apply(this, param)
    }
}
```

案例2🌰

```html
<body>
    <input id="text" onkeyup="changeInput(event)" />
    <p id="result"></p>
    <script>
        let text = document.querySelector('#text')
        let result = document.querySelector('#result')
        function changeInput (event) {
            // console.log(event.target.value)
            debouncePostValue(event.target.value)
        }
        function debounce (func, await = 500) {
            let timer = null;
            return function (...param) {
                if (timer) clearTimeout(timer)
                let callNow = !timer
                timer = setTimeout(() => { // 在await之后将定时器清除
                    timer = null
                }, await)
                if (callNow) func.apply(this, param)
            }
        }
        const debouncePostValue = debounce(postValue)
        function postValue (value) {
            result.innerHTML = value
        }
    </script>
</body>
```



### 终极防抖

若是将是否立即执行作为一个参数传入函数中，就可以合成最终的一个防抖函数。这里我用到的是[yck](https://juejin.im/user/574f8d8d2e958a005fd4edac)大佬提供的一种防抖写法：

```javascript
/**
 * 防抖函数，返回函数连续调用时，空闲时间必须大于或等于 wait，func 才会执行
 *
 * @param  {function} func        回调函数
 * @param  {number}   wait        表示时间窗口的间隔
 * @param  {boolean}  immediate   设置为ture时，是否立即调用函数
 * @return {function}             返回客户调用函数
 */
function debounce (func, wait = 50, immediate = true) {
  let timer, context, args

  // 延迟执行函数
  const later = () => setTimeout(() => {
    // 延迟函数执行完毕，清空缓存的定时器序号
    timer = null
    // 延迟执行的情况下，函数会在延迟函数中执行
    // 使用到之前缓存的参数和上下文
    if (!immediate) {
      func.apply(context, args)
      context = args = null
    }
  }, wait)

  // 这里返回的函数是每次实际调用的函数
  return function(...params) {
    // 如果没有创建延迟执行函数（later），就创建一个
    if (!timer) {
      timer = later()
      // 如果是立即执行，调用函数
      // 否则缓存参数和调用上下文
      if (immediate) {
        func.apply(this, params)
      } else {
        context = this
        args = params
      }
    // 如果已有延迟执行函数（later），调用的时候清除原来的并重新设定一个
    // 这样做延迟函数会重新计时
    } else {
      clearTimeout(timer)
      timer = later()
    }
  }
}
```

下面有一个案例是演示了如何调用，并传递参数：

```html
<body>
    <button onclick="clickBtn()">点击</button>
    <p id="result"></p>
    <script>
        let num = 0
        function clickBtn () {
            debouncePostValue('点击了')
        }
        /**
         * 防抖函数，返回函数连续调用时，空闲时间必须大于或等于 wait，func 才会执行
         *
         * @param  {function} func        回调函数
         * @param  {number}   wait        表示时间窗口的间隔
         * @param  {boolean}  immediate   设置为ture时，是否立即调用函数
         * @return {function}             返回客户调用函数
         */
        function debounce(func, wait = 50, immediate = true) {
            let timer, context, args

            // 延迟执行函数
            const later = () => setTimeout(() => {
                // 延迟函数执行完毕，清空缓存的定时器序号
                timer = null
                // 延迟执行的情况下，函数会在延迟函数中执行
                // 使用到之前缓存的参数和上下文
                if (!immediate) {
                    func.apply(context, args)
                    context = args = null
                }
            }, wait)

            // 这里返回的函数是每次实际调用的函数
            return function (...params) {
                // 如果没有创建延迟执行函数（later），就创建一个
                if (!timer) {
                    timer = later()
                    // 如果是立即执行，调用函数
                    // 否则缓存参数和调用上下文
                    if (immediate) {
                        func.apply(this, params)
                    } else {
                        context = this
                        args = params
                    }
                    // 如果已有延迟执行函数（later），调用的时候清除原来的并重新设定一个
                    // 这样做延迟函数会重新计时
                } else {
                    clearTimeout(timer)
                    timer = later()
                }
            }
        }
        const debouncePostValue = debounce(postValue, 500, true)
        function postValue(laterParam) {
            result.innerHTML = `${laterParam}:${++num}`
        }
    </script>
</body>
```


## 节流

防抖动和节流本质是不一样的。防抖动是将多次执行变为最后一次执行，节流是将多次执行变成每隔一段时间执行，我们来直接看代码：

### 简易版：
```javascript
// 节流 时间戳版
function throttle1 (fn, delay) {
    let prev = 0
    return function () {
        const self = this
        const args = arguments
        const now = +new Date()
        if (now - prev > delay) {
            fn.apply(self, args)
            prev = now
        }
    }
}

// 节流 定时器版
function throttle2 (fn, wait) {
    let timer
    return function () {
        const self = this
        const args = arguments
        if (!timer) {
            timer = setTimeout(() => {
                fn.apply(self, args)
                timer = null
            }, wait);
        }
    }
}
```

### yck大佬的终极实现
```javascript
/**
 * underscore 节流函数，返回函数连续调用时，func 执行频率限定为 次 / wait
 *
 * @param  {function}   func      回调函数
 * @param  {number}     wait      表示时间窗口的间隔
 * @param  {object}     options   如果想忽略开始函数的的调用，传入{leading: false}。
 *                                如果想忽略结尾函数的调用，传入{trailing: false}
 *                                两者不能共存，否则函数不能执行
 * @return {function}             返回客户调用函数
 */
_.throttle = function(func, wait, options) {
    var context, args, result;
    var timeout = null;
    // 之前的时间戳
    var previous = 0;
    // 如果 options 没传则设为空对象
    if (!options) options = {};
    // 定时器回调函数
    var later = function() {
      // 如果设置了 leading，就将 previous 设为 0
      // 用于下面函数的第一个 if 判断
      previous = options.leading === false ? 0 : _.now();
      // 置空一是为了防止内存泄漏，二是为了下面的定时器判断
      timeout = null;
      result = func.apply(context, args);
      if (!timeout) context = args = null;
    };
    return function() {
      // 获得当前时间戳
      var now = _.now();
      // 首次进入前者肯定为 true
	  // 如果需要第一次不执行函数
	  // 就将上次时间戳设为当前的
      // 这样在接下来计算 remaining 的值时会大于0
      if (!previous && options.leading === false) previous = now;
      // 计算剩余时间
      var remaining = wait - (now - previous);
      context = this;
      args = arguments;
      // 如果当前调用已经大于上次调用时间 + wait
      // 或者用户手动调了时间
 	  // 如果设置了 trailing，只会进入这个条件
	  // 如果没有设置 leading，那么第一次会进入这个条件
	  // 还有一点，你可能会觉得开启了定时器那么应该不会进入这个 if 条件了
	  // 其实还是会进入的，因为定时器的延时
	  // 并不是准确的时间，很可能你设置了2秒
	  // 但是他需要2.2秒才触发，这时候就会进入这个条件
      if (remaining <= 0 || remaining > wait) {
        // 如果存在定时器就清理掉否则会调用二次回调
        if (timeout) {
          clearTimeout(timeout);
          timeout = null;
        }
        previous = now;
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      } else if (!timeout && options.trailing !== false) {
        // 判断是否设置了定时器和 trailing
	    // 没有的话就开启一个定时器
        // 并且不能不能同时设置 leading 和 trailing
        timeout = setTimeout(later, remaining);
      }
      return result;
    };
  };
```
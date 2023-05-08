---
title: JS进阶之偏函数和柯里化
date: 2018-12-25 22:08:29
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

在上一篇文章中我们主要介绍了函数的一些基本功能和结构，以及介绍了一些实用的小技巧。这些都是为了后面一步一步入门打下好的基础。因为函数式编程并不是一个看看文档就能很好掌握的东西，它需要你集合实际例子然后理解每一步为什么要这样，如果你只是想粗略的看看，不去思考，相信我，后面的案例你会感觉特别跳，特别绕(开始学习时我就是这样)。

在这一章中，我会针对函数式编程的另一个重点：**函数的输入**来做讲解和案例分析，个人建议：打开你的`vscode`，关上文档，把案例敲上一遍，需要的时候把每一步做个对比，确保自己是真的理解它们。

## 偏函数

先来看一个大家都很熟悉的函数：

1. 一个`ajax`函数，第一个参数为请求的`API`地址，第二个为请求的参数，第三个是请求成功之后的回调函数。

```javascript
function ajax (url, data, callback) {
	// ...
}
```

2. 现在如果你已经很确定一个`API`地址，此外只是需要等待另外两个参数的时候，比如获取用户信息和获取订单详情的请求：

```javascript
function getUser (data, cb) {
	ajax('/api/user', data, cb)
}
function getOrder (data, cb) {
	ajax('api/order', data, cb)
}
```

3. 现在如果你已经很确定一个`API`地址，同时已经很确定请求的参数(比如用户的`id`)，此外只需要等待另一个参数的时候：

```javascript
function getCurrentUser (cb) {
	getUser({ userId: 1 }, cb)
}
function getCurrentOrder (cb) {
	getUser({ orderId: 1 }, cb)
}
```

不知道大家发现了没，从第一步到第三步，每过一步，函数的参数就少一个，直到最后只需要传递一个`cb`。

用一句话来说明发生的事情：`getUser(data, cb)`是`ajax(url, data, cb)`的**偏函数(partially-applied functions)**。

(注意⚠️：前方高能！)

关于该模式更正式的说法是：偏函数严格来讲是一个减少函数参数个数（arity）的过程；这里的参数个数指的是希望传入的形参的数量。我们通过 `getUser(..)` 把原函数 `ajax(..)` 的参数个数从 3 个减少到了 2 个。

### partial函数

在上面的例子中，`getCurrentUser(cb)`和`getCurrentOrder(cb)`的模式其实很想，我们可以来定一个`partial()`实用函数：

```javascript
function partial (fn, ...prestArgs) {
	return function partiallyApplied (...laterArgs) {
		return fn(...prestArgs, ...laterArgs)
	}
}
```

`partial`函数接受一个`fn`函数，和若干个参数`…prestArgs`。

它返回的是另一个函数`partiallyApplied()`函数，这个函数也接受若干个参数`…laterArgs`，并返回`partial`函数传递进来`fn`函数。

返回的`fn`函数会将`partial`和`partiallyApplied`中的参数都接收过去。

(这个实用函数我至少敲了3遍...)

好吧，我们还是来看看我参考资料的原版本是怎么描述这个实用函数的吧，感觉它说的也比较清晰：

> `partial(..)` 函数接收 `fn` 参数，来表示被我们偏应用实参（partially apply）的函数。接着，`fn` 形参之后，`presetArgs` 数组收集了后面传入的实参，保存起来稍后使用。
>
> 我们创建并 `return` 了一个新的内部函数（为了清晰明了，我们把它命名为`partiallyApplied(..)`），该函数中，`laterArgs` 数组收集了全部实参。
>
> 你注意到在内部函数中的 `fn` 和 `presetArgs` 引用了吗？他们是怎么如何工作的？在函数 `partial(..)` 结束运行后，内部函数为何还能访问 `fn` 和 `presetArgs` 引用？你答对了，就是因为**闭包**！内部函数 `partiallyApplied(..)` 封闭（closes over）了 `fn` 和 `presetArgs` 变量，所以无论该函数在哪里运行，在 `partial(..)` 函数运行后我们仍然可以访问这些变量。所以理解闭包是多么的重要！
>
> 当 `partiallyApplied(..)` 函数稍后在某处执行时，该函数使用被闭包作用（closed over）的 `fn` 引用来执行原函数，首先传入（被闭包作用的）`presetArgs` 数组中所有的偏应用（partial application）实参，然后再进一步传入 `laterArgs` 数组中的实参。

当然你也可以用更便捷的箭头函数语法来重写上面的函数：

```javascript
var partial = (fn, ...presetArgs) => 
														(...laterArgs) => 
																fn(...prestArgs, ...laterArgs);
```

**优点**：更加简洁，甚至代码稀少。

**缺点**：函数会变成匿名函数，可读性上失去益处，此外，由于作用域边界变得模糊，我们会更加难以辩认闭包。

不过是否采用箭头函数都是你的个人喜好。

### ajax案例

1. 介绍完上面的函数，我们现在可以用`partial`实用函数来制造这些之前提及的偏函数：

```javascript
// example1
function partial (fn, ...prestArgs) {
	return function partiallyApplied (...laterArgs) {
		return fn(...prestArgs, ...laterArgs)
	}
}

var getUser = partial(ajax, '/api/user')

var getOrder = partial(ajax, '/api/order')
```

不知道大家脑中是否有`getUser` 函数的外形和内在，它其实就相当于这样：

```javascript
var getUser = partial(ajax, '/api/user')
// 相当于=>
var getUser = function partailApplication (...laterArgs) {
  return ajax('/api/user', ...laterArgs)
}
```

2. 我相信大家已经知道怎样用`partial`来写`getUser`函数了

那么再进一层，`getCurrentuser`函数可以怎么写呢？

```javascript
// example2
var getCurrentUser = partial(ajax, '/api/user', { userId: 1 })
```

哈哈，看到这里你是否想到了还能用案例1中的`getUser`和`partial`配合：

```javascript
// example3
var getCurrentUser = partial(getUser, { userId: 1 })
```

过程是这样的：

```javascript
function ajax (url, data, callback) {
	// ...
}

function partial (fn, ...prestArgs) {
	return function partiallyApplied (...laterArgs) {
		return fn(...prestArgs, ...laterArgs)
	}
}

var getUser = partial(ajax, '/api/user')

var getCurrentUser = partial(getUser, { userId: 1 })
```

我们可以像案例2一样通过指定`url`和`data`两个实参来定义`getCurrentUser(...)`函数。

也可以像案例3将`getCurrentUser(…)`函数定义成`getUser(…)`的偏应用，该偏应用仅指定一个附加的 `data` 实参。

> 案例3的函数包含了一个额外的函数包装层。这看起来有些奇怪而且多余，但对于你真正要适应的函数式编程来说，这仅仅是它的冰山一角。随着本文的继续深入，我们将会把许多函数互相包装起来。记住，这就是**函数**式编程！

### add案例

理解了上面的一个案例之后，我们再来看下面的案例应该就会变得非常简单了：

这是一个计算返回两数之和的函数：

```javascript
function add (x, y) {
	return x + y
}
```

现在我们有一个数组，要给数组中的每一项都固定加上一个数`3`，也许你想到了可以用`JS`中的`map`来写：

```javascript
var arr = [1, 2, 3, 4]
var arr2 = arr.map(function adder (val) => {
	return add(3, val)
})
```

`map`中执行的事情其实也是返回一个函数`add`的计算结果，那么我们就可以用`partial`函数来写它：

```javascript
// example4
var arr2 = arr.map(partial(add, 3))
```

**注意：** 如果你没见过 `map(..)` ，别担心，我会在后面的部分详细介绍它。目前你只需要知道它用来循环遍历（loop over）一个数组，在遍历过程中调用函数产出新值并存到新的数组中。



## 柯里化

我们来看一个跟偏应用类似的技术，该技术将一个期望接收多个实参的函数拆解成连续的链式函数（chained functions），每个链式函数接收单一实参（实参个数：1）并返回另一个接收下一个实参的函数。

这就是柯里化（currying）技术。

还记得前面的`ajax`函数吗？

```javascript
function ajax (url, data, callback) {
	// ...
}
```

现在想象一下我们已经创建了一个`ajax(…)`的柯里化版本：

```javascript
curriedAjax('/api/user')
						({ userId: 1 })
							( function foundUser(user) { ... } )	
```

我们将三次调用分别拆解开来，这也许有助于我们理解整个过程：

```javascript
var userFetcher = curriedAjax('/api/user')
var getCurrentUser = userFetcher({ userId: 1 })
getCurrentUser( function foundUser(user){ /* .. */ } )
```

可以看到`curriedAjax`函数在每次调用的时候只接收一个实参，而不是一次性接收所有实参（像 `ajax(..)` 那样），也不是先传部分实参再传剩余部分实参（借助 `partial(..)` 函数）。

**柯里化和偏应用进行对比**：

相同点：

- 每个类似偏应用的连续柯里化调用都把另一个实参应用到原函数，一直到所有实参传递完毕。

不同点：

- 柯里化会明确地返回一个期望**只接收下一个实参** `data` 的函数，而偏应用是能接收所有的剩余参数。

### curry函数

下面我们来看看如何定义一个用来柯里化的实用函数：

```javascript
function curry(fn, arity = fn.length) {
  return (function nextCurried(prevArgs) {
    return function curried(nextArg) {
      var args = prevArgs.concat([nextArg])
      if (args.length >= arity) {
        return fn(...args)
      } else {
        return nextCurried(args)
      }
    }
  })([])
}
```

`ES6`箭头函数版本：

```javascript
var curry = (fn, arity = fn.length, nextCurried) => 
  (nextCurried = prevArgs => {
    nextArg => {
      var args = prevArgs.concat( [nextArg] );
      if (args.length >= arity) {
        return fn( ...args );
      }
      else {
        return nextCurried( args );
      }
    }
  })([])
```



> 此处的实现方式是把空数组 `[]` 当作 `prevArgs` 的初始实参集合，并且将每次接收到的 `nextArg` 同 `prevArgs` 连接成 `args` 数组。当 `args.length` 小于 `arity`（原函数 `fn(..)` 被定义和期望的形参数量）时，返回另一个 `curried(..)`函数（译者注：这里指代 `nextCurried(..)` 返回的函数）用来接收下一个 `nextArg` 实参，与此同时将 `args` 实参集合作为唯一的 `prevArgs` 参数传入 `nextCurried(..)` 函数。一旦我们收集了足够长度的 `args` 数组，就用这些实参触发原函数 `fn(..)`。
>
> 默认地，我们的实现方案基于下面的条件：在拿到原函数期望的全部实参之前，我们能够通过检查将要被柯里化的函数的 `length` 属性来得知柯里化需要迭代多少次。
>
> 假如你将该版本的 `curry(..)` 函数用在一个 `length` 属性不明确的函数上 —— 函数的形参声明包含默认形参值、形参解构，或者它是可变参数函数，用 `...args` 当形参；参考第 2 章 —— 你将要传入 `arity` 参数（作为 `curry(..)` 的第二个形参）来确保 `curry(..)` 函数的正常运行。

### ajax案例

我们用 `curry(..)` 函数来实现此前的 `ajax(..)` 例子：

```javascript
var curriedAjax = curry( ajax )
var userFetcher = curriedAjax('/api/user')
var getCurrentUser = userFetcher({ userId: 1 })
getCurrentUser( function foundUser(user){ /* .. */ } )
```

可以看到在每次函数调用的时候都会新增一个实参，最终给原函数`ajax`使用，直到收齐了三个实参并执行`ajax`函数为止。

### add案例

现在我们还可以来回顾一下在`partial`中用到的例子：

```javascript
var arr = [1, 2, 3, 4]
var arr2 = arr.map( partial(add, 3) )
```

由于柯里化是和偏应用相似的，所以我们可以用几乎相同的方式以柯里化来完成那个例子。

```javascript
var arr2 = arr.map( curry( add )( 3 ) );
// [4,5,6,7,8]
```

`partial(add,3)` 和 `curry(add)(3)` 两者有什么不同呢？为什么你会选 `curry(..)` 而不是偏函数呢？当你先得知 `add(..)` 是将要被调整的函数，但如果这个时候并不能确定 `3` 这个值，柯里化可能会起作用：

```javascript
var adder = curry( add );

// later
[1,2,3,4,5].map( adder( 3 ) );
// [4,5,6,7,8]
```

### sum案例

下面这个案例，是将一个列表的数字相加：

```javascript
function sum(...args) {
	var sum = 0;
	for (let i = 0; i < args.length; i++) {
		sum += args[i];
	}
	return sum;
}
```

普通调用：

```javascript
sum(1, 2, 3, 4, 5)
// 15
```

柯里化调用

```javascript
// (5 用来指定需要链式调用的次数)
var curriedSum = curry( sum, 5 )
curriedSum( 1 )( 2 )( 3 )( 4 )( 5 ) // 15
```

柯里化调用的好处：

- 每次函数调用传入一个实参，并生成另一个特定性更强的函数，之后我们可以在程序中获取并使用**那个**新函数。
- 偏应用则是预先指定所有将被偏应用的实参，产出一个等待接收剩下所有实参的函数。

## 柯里化和偏应用有什么用？

柯里化和偏应用这两种风格的签名都比普通的函数要奇怪很多，那么为什么要用这么奇怪的方式去构造那些函数呢？主要是有这么几个方面：

- 使用柯里化和偏应用可以将指定分离实参的时机和地方独立开来，传统函数是需要预先确定所有实参的。
- 当函数只有一个形参时，我们能够比较容易地组合它们

## 柯里化多个参数

在上面介绍的函数柯里化中，我们知道，它在每次调用的时候只支持传入一个实参。这样的柯里化我们可以称之为“严格柯里化”。

其实在大多数流行的`JavaScript`函数式编程都使用了一种不严格的柯里化(loose currying)。

也就是说，往往 JS 柯里化实用函数会允许你在每次柯里化调用中指定多个实参，如在上面提到的`sum`函数，我们使用严格柯里化需要调用5次，但在松散柯里化我们可以这样：

```javascript
var curriedSum = looseCurry(sum, 5)
curriedSum(1)(2, 3)(4, 5)
```

相比于严格的柯里化，语法上我们节省了`()`的使用，并且把五次函数调用减少成三次，间接提高了性能。

**注意：** 松散柯里化**允许**你传入超过形参数量（arity，原函数确认或指定的形参数量）的实参。如果你将函数的参数设计成可配的或变化的，那么松散柯里化将会有利于你。

现在我们可以将之前的柯里化函数调整一下，使其适应这种常见的更松散的定义：

```javascript
function looseCurry(fn, arity = fn.length) {
  return (function nextCurried(prevArgs) {
    return function curried(...nextArgs) {
      var args = prevArgs.concat(nextArgs);
      if (args.length >= arity) {
        return fn(...args);
      } else {
        return nextCurried(args);
      }
    };
  })([]);
}
```

`ES6`版本：

```javascript
var looseCurry = (fn, arity = fn.length, nextCurried) =>
  (nextCurried = prevArgs => (...nextArg) => {
    var args = prevArgs.concat(nextArg);
    if (args.length >= arity) {
      return fn(...args);
    } else {
      return nextCurried(args);
    }
  })([]);
```

## 反柯里化

你也会遇到这种情况：拿到一个柯里化后的函数，却想要它柯里化之前的版本 —— 这本质上就是想将类似 `f(1)(2)(3)` 的函数变回类似 `g(1,2,3)` 的函数。

处理这个需求的标准实用函数通常被叫作 `uncurry(..)`：

```javascript
function uncurry(fn) {
	return function uncurried(...args){
		var ret = fn;

		for (let i = 0; i < args.length; i++) {
			ret = ret( args[i] );
		}

		return ret;
	};
}
```

`ES6`版本

```javascript
var uncurry = fn => 
	uncurried = (...args) => {
		var ret = fn
		for (let i = 0; i < args.length; i++) {
			ret = ret( args[i] )
		}
		return ret
	}
```

使用反柯里化后，可以让我们函数的传参形式变为柯里化之前的形式：

```javascript
// example5
function sum(...args) {
	var sum = 0;
	for (let i = 0; i < args.length; i++) {
		sum += args[i];
	}
	return sum;
}

var curriedSum = curry( sum, 5 );
var uncurriedSum = uncurry( curriedSum );

curriedSum( 1 )( 2 )( 3 )( 4 )( 5 );		// 15
uncurriedSum( 1, 2, 3, 4, 5 );				// 15
```

**注意⚠️**

但不要以为使用了反柯里化之后的函数会和原函数的行为完全一样(也就是uncurry(curry(fn))和 fn )，虽然在某些库中，反柯里化使函数变成和原函数（译者注：这里的原函数指柯里化之前的函数）类似的函数。

但是凡事皆有例外，例如我们上面的案例5，采用反柯里化之后，如果你少传了实参，就会得到一个仍然在等待传入更多实参的部分柯里化函数。我们在下面的代码中说明这个怪异行为。

```javascript
uncurriedSum( 1, 2, 3, 4, 5 ) // 15
uncurriedSum( 1, 2, 3 )( 4, 5 ) // 15
```

这两种传参方式都会得到相同的结果。

`uncurry()` 函数最为常见的作用对象很可能并不是人为生成的柯里化函数（例如上文所示），而是某些操作所产生的已经被柯里化了的结果函数。我会在后面关于 “无形参风格” 的讨论中阐述这种应用场景。



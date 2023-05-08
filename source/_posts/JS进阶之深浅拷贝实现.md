---
title: JS进阶之深浅拷贝实现
date: 2018-11-08 09:55:43
type: "js-base"
tag: js
description:
keywords:
top_img:
mathjax:
katex:
aside:
---


### 一、问

#### 1. 有哪些属于浅拷贝?

1. `Object.assign()`方法
2. 展开算法`Spread` (`{...obj}`)
3. 操作数组的方法`slice()、concat()`



#### 2. 有哪些深拷贝的方式?

1. 比较流氓式的方式: `JSON.parse(JSON.stringify(object))`
2. 自己实现一个深拷贝的方法
3. 使用一些工具库里的方法, 比如`lodash`里的`cloneDeep()`, `jquery`里的`extend()`.



#### 3. JSON.parse(JSON.stringify(object))的缺点

1. 会忽略属性值为`undefined`、`symbol`、函数的这三种情况, 也就是不进行拷贝
2. 不能解决循环引用问题
3. 不能正确的处理`new Date`
4. 不能处理正则(为正则的时候, 拷贝过去为一个空对象`{}`)



### 二、实现

#### 1. 手写一个`Object.assign()`函数实现

**思路:**

1. 判断原生`Object`是否支持`assign`, 若是不存在的话则使用`Object.defineProperty`方法将该函数绑定到`Object` 中
2. 判断传入的目标参数是否正确, 是否为`null`
3. 使用`Object()`将目标参数转为对象, 并保存为`to`, 最终要返回这个`to`
4. 使用 `for..in` 循环遍历出所有可枚举的自有属性。并复制给新的目标对象（使用 `hasOwnProperty` 获取自有属性，即非原型链上的属性）
5. 在设计的时候需要在让该方法在严格模式下进行

```javascript
function createAssign2() {
    // 1. 判断是否存在assign2
    if (typeof Object.assign2 !== 'function') {
        // 2. 给 Object 中添加属性
        Object.defineProperty(Object, 'assign2', {
            value: function(target) {
                // 4. 开启严格模式
                "use strict";
                // 5. 判断目标对象是否为 null
                if (target == null) {
                    throw new Error("Cannot convert undefined or null to object")
                }
                // 6. 使用 Object() 包裹
                var to = Object(target);
                // 7. 遍历后面传入的所有对象
                for (var i = 1; i < arguments.length; i++) {
                    var nextSource = arguments[i];
                    // 8. 使用 for...in 查找所有的属性(包括原型链中的属性)
                    for (var nextKey in nextSource) {
                        // 9. 只考虑自身的属性
                        if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
                            to[nextKey] = nextSource[nextKey];
                        }
                    }
                }
                // 10. 返回to
                return to;
            },
            writable: true,
            configurable: true
        })
    }
}
createAssign2();
```

**注**:

1. 为什么要在严格模式下?

那是因为使用原生的`Object.assign()`时, 若是有以下情况:

```javascript
var newObj = Object.assign('123', '456', { name: 'ZhiWeiLi' });
// or
var newObj = Object.assign('123', { 0: '4' });
```

上面两种情况都会报错:

```javascript
Cannot assign to read only property '0' of object '[object String]'
```

但如果是下面的情况就不会:

```javascript
var newObj = Object.assign(123, '456'); // Number{0:"4", 1:"5", 2:"6" }
// or
var newObj = Object.assign('123', 456) // String{0: "1", 1: "2", 2: "3"}
```

看出什么了吗?

如果参数中同时存在两个字符串, 比如`"123"和"456"`就会报错, 或者存在一个字符串和一个带有`0`属性的对象, 也会报错.

那是因为字符串类型的数据的**可写属性writable**是`false`:

```javascript
var str = 'abc';
console.log(Object.getOwnPropertyDescriptor(str, "0"));
// { 
//   value: 'a',
//   writable: false, // 可写属性
//   enumerable: true,
//   configurable: false 
// }
```

而平常我们使用**下标修改**的方式直接修改是**不会报错的**, 只会**修改不成功**:

```javascript
var str = "abc";
str[0] = "d";
console.log(str[0]); // "a" 还是"a", 证明没有修改成功
```

但是在**严格模式**下就会**报错**了:

```javascript
"use strict";
var str = "abc";
str[0] = "d";
// Cannot assign to read only property '0' of object '[object String]'
```

换句话说: 在**严格模式下**, 如果你试图修改一个`writable: false`的属性的时候就会报错.

所以若是`assign()`方法的第一个参数的数据类型是**非空的字符串**, 而后面也有**非空的字符串**或者**带有`0`属性的对象**时, 就会报错, 因此我们在设计`assign2`的时候, 需要在严格模式下进行.



2. 为什么需要有`var to = Object(target)`这一步?

因为如果第一个参数`target`为基本数据类型(`string、number、symbol`)时, 需要将其包装为对象, 如果没有`Object(target)`这一步的话, 就会报错了:

```javascript
Uncaught TypeError: Cannot create property 'obj' on string '123'
```

而如果第一个参数是这种基本数据类型的时候, 生成的对象会是这样:

```javascript
var newObj = Object.assign("123", { name: "ZhiWeiLi" });
// String{ 0: "1", 1: "2", 2: "3", name: "ZhiWeiLi"}
```

前面的`String`表示新生成的对象的`__proto__`为`String`对象(而非`Object`对象), 而且其中有一个名为`[[PrimitiveValue]]`的属性为`"123"`.



**测试代码：**

```javascript
createAssign2();
var obj = {
  name: 'obj',
  colors: ['white', 'black']
}
var obj2 = Object.assign2({}, obj, { sex: 'boy' })
obj2.name = 'obj2'
obj2.colors.push('yellow')
console.log(obj) // { name: 'obj', colors: ['white', 'black', 'yellow'], sex: 'boy' }
console.log(obj2) // { name: 'obj2', colors: ['white', 'black', 'yellow'], sex: 'boy' }
```



#### 2. 手写一个深拷贝方法

**思路**:

1. 对传入的参数做校验, 不是对象类型则返回参数本身
2. 校验的方式, 可以使用`Object.prototype.toString.call(obj)`的方式来判断
3. 需要考虑是数组还是对象
4. 需要考虑循环引用问题(可以用`WeakMap`或者数组来解决)
5. 需要判断每一个属性的类型, 若是`Object`和`Array`的时候, 要进行递归处理


**代码**:

```javascript
// 判断类型是不是对象或者数组
function isObject(object) {
    let type = Object.prototype.toString.call(object).slice(8, -1);
    return type === 'Object' || type === 'Array';
}
function cloneDeep(source, hash = new WeakMap()) {
    // 1. 判断不是数组或者对象的时候则返回其本身
    if (!isObject(source)) return source
    // 2. 通过WeakMap解决循环引用问题
    if (hash.has(source)) return hash.get(source);
    // 3. 判断是否是数组
    var target = Array.isArray(source) ? [...source] : {...source};
    hash.set(source, target);
    // 4. 利用`Reflect.ownKeys()`获取target本身的属性(非原型链上的属性), 且包含了symbol类型的属性
    Reflect.ownKeys(target).forEach(key => {
        // 5. 再次判断是否是对象类型
        if (isObject(source[key])) {
            target[key] = cloneDeep(source[key], hash);
        } else {
            target[key] = source[key];
        }
    })
    // 6. 返回target
    return target;
}
```

**测试代码**:

```javascript
var obj = {
    a: undefined,
    b: null,
    c: Symbol('c'),
    d: function() {
        console.log(d)
    },
    e: new Date(),
    f: /^123/,
    g: {
        name: 'objName'
    }
}
obj[Symbol("h")] = "localH"; // symbol类型
obj.a = obj.g; // 循环引用
obj.g.sex = obj.a

var newObj = cloneDeep(obj);
obj.g.name = 'modifyName';
console.log(obj)
console.log(newObj)
```

**注**:

1. 为什么判断是否是对象需要这样写?

这里我判断是否是对象是通过`Object.prototype.toString.call(object)`的方式,

我看有些教材是采用:

```javascript
function isObject (object) {
	return typeof object === 'object' && object !== null;
}
```

但是如果属性值是`Date、RegExp`的时候, 也会被判断为`true`了, 这样就走了递归路线, 导致这种类型的值拷贝失败...



2. 为什么遍历属性要这样写?

这里我遍历属性的方式是:

```javascript
Reflect.ownKeys(target).forEach(key => {})
```

采用此方式的优点是:

- 只遍历元素自身的属性, 不会遍历原型链上的
- 可以遍历出类型为`symbol`类型的属性
- 它与`Object.keys()`相似, 但是它不会受`enumerable`影响



3. 使用`for...in...`遍历属性有什么问题?

采用`for...in...`的形式的话其实也可以, 不过你得多做几个处理了, 因为`for...in...`的形式有以下两个问题:

- 会把原型链上的属性也遍历出来, 因此你得这样判断:

```javascript
for (var key in target) {
  if (Object.prototype.hasOwnProperty.call(source, key)) {
    // 这里就过滤掉了原型链上的属性
  }
}
```

- 遍历不了`symbol`类型的属性

如果你想要获取类型为`symbol`的属性可以采用下面这个方法:

```javascript
var obj = {};
obj[Symbol("h")] = "localH";

console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(h)]
```

而我这里用的`Reflect.ownKeys()`的方法其实就等同于下面的写法:

```javascript
Object.getOwnPropertyNames(obj).concat(Object.getOwnPropertySymbols(obj));
```

此时兼容的写法就是:

```javascript
// 这里获取到了所有symbol类型的属性
var symbolKeys = Object.getOwnPropertySymbols(target);
if (symbolKeys.length > 0) {
	symBolKeys.forEach(symKey = > {
		if (isObject(source[symKey])) {
			target[symKey] = cloneDeep(source[symKey], hash);
		} else {
			target[symKey] = source[symKey];
		}
	})
}
// 下面这个 for...in... 是遍历非symbol类型的属性
for (var key in target) {
  if (Object.prototype.hasOwnProperty.call(source, key)) {
    if (isObject(source[key])) {
			target[key] = cloneDeep(source[key], hash);
		} else {
			target[key] = source[key];
		}
  }
}
```

这样其实也可以, 不过感觉没有必要...



**测试代码：**

```javascript
var obj = {
  name: 'obj',
  colors: ['white', 'black'],
  persons: [{ name: 'p1' }, { name: 'p2' }]
}
var obj2 = cloneDeep(obj)
obj2.name = 'obj2'
obj2.colors.push('yellow')
console.log(obj) // { name: 'obj', colors: ['white', 'black'], persons: [...] }
console.log(obj2) // { name: 'obj2', colors: ['white', 'black', 'yellow'], persons: [...] }
```



#### 3. 如何解决Object.assign不能拷贝set和get的问题？

**问题产生原因**：

```javascript
const source = {
  set foo (value) {
    console.log(value)
  },
  get bar () {
    return 'ZhiWeiLi'
  }
}
const target1 = {}
Object.assign(target1, source)
console.log(Object.getOwnPropertyDescriptor(target1, 'foo'))
```

结果为：

```javascript
{value: undefined, writable: true, enumerable: true, configurable: true}
```

这里的`foo`是一个`set`属性，但是在进行`Object.assign()`调用的时候确不能正确的拷贝，也就是获取到的`value`是为`undefined`。

**解决办法**：

可以使用`ES8`中的`getOwnPropertyDescriptors()`配合`Object.defineProperty()`：

```javascript
const source = {
  set foo (value) {
    console.log(value)
  },
  get bar () {
    return 'ZhiWeiLi'
  }
}
const target1 = {}
// Object.assign(target1, source)
// console.log(Object.getOwnPropertyDescriptor(target1, 'foo'))
Object.defineProperties(target1, Object.getOwnPropertyDescriptors(source))
console.log(Object.getOwnPropertyDescriptor(target1, 'foo'))
```

结果为：

```javascript
{get: undefined, enumerable: true, configurable: true, set: ƒ}
```
****


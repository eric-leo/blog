---
title: ES6之常用知识总结
date: 2019-05-08 20:25:44
type: "js-base"
tag: es6
description:
keywords:
top_img:
mathjax:
katex:
aside:
---


## ECMAScript6

ECMAScript简称就是ES,你可以把它看成是一套标准,JavaScript就是实施了这套标准的一门语言,现在主流浏览器使用的是ECMAScript5。

[https://babeljs.io/repl](http://babeljs.io/repl)

###  作用域变量
作用域就是一个变量的作用范围。也就是你声明一个变量以后,这个变量可以在什么场合下使用 以前的JavaScript只有全局作用域，还有一个函数作用域

#### var的问题

1.var没有块级作用域，定义后在当前闭包中都可以访问，如果变量名重复，就会覆盖前面定义的变量，并且也有可能被其他人更改
```javascript
if (true) {
     var a = "a"; // 期望a是某一个值
 }
console.log(a);
```

2.var在for循环标记变量共享，一般在循环中使用的i会被共享，其本质上也是由于没有块级作用域造成的

```javascript
for (var i = 0; i < 3; i++) {
     setTimeout(function () {
         alert(i);
     }, 0);
 }
```

> 结果
> 弹窗三次 3

#### 块级作用域

在用var定义变量的时候，变量是通过闭包进行隔离的，现在用了let，不仅仅可以通过闭包隔离，还增加了一些块级作用域隔离。 块级作用用一组大括号定义一个块,使用 let 定义的变量在大括号的外面是访问不到的

##### 实现块级作用域

```javascript
    let name = 'zhiweili';
}
console.log(name);// ReferenceError: name is not defined
```

#### 不会污染全局对象

```javascript
if(true){
    let name = 'zhiweili';
}
console.log(window.name); // undefined
```

#### for循环中也可以使用i

```javascript
// 嵌套循环不会相互影响
for (let i = 0; i < 3; i++) {
    console.log("out", i);
    for (let i = 0; i < 2; i++) {
        console.log("in", i);
    }
}
```

> 结果 out 0 in 0 in 1 out 1 in 0 in 1 out 2 in 0 in 1

#### 重复定义会报错

```javascript
if(true){
    let a = 1;
    let a = 2; //Identifier 'a' has already been declared
}
```

#### 不存在变量的预解释

```javascript
for (let i = 0; i < 2; i ++){
    console.log('inner',i);
    let i = 100;
}
```

> 结果 i is not defined

####  闭包的新写法

以前

```javascript
;(function () {

})();
```

现在

```javascript
{
}
```

### 常量

使用const我们可以去声明一个常量，常量一旦赋值就不能再修改了

#### 常量不能重新赋值

```javascript
const MY_NAME = 'zhiweili';
MY_NAME = 'zhiweili2';//Assignment to constant variable
```

#### 变量值可改变

> 注意const限制的是不能给变量重新赋值，而变量的值本身是可以改变的,下面的操作是可以的

```javascript
const names = ['zhiweili1'];
names.push('zhiweili2');
console.log(names);
```

#### 不同的块级作用域可以多次定义

```javascript
{
    const A = "A";
    console.log(A)
}
{
    const A = "B";
    console.log(A)
}
console.log(A)
```

> 结果 A B 0

### 解构

#### 解析数组

解构意思就是分解一个东西的结构,可以用一种类似数组的方式定义N个变量，可以将一个数组中的值按照规则赋值过去。

```javascript
var [name,age] = ['zhiweili',8];
console.log(name,age);
```

#### 嵌套赋值

```javascript
 let [x, [y], z] = [1, [2.1, 2.2]];
    console.log(x, y, z);

    let [x, [y,z]] = [1, [2.1, 2.2]];
    console.log(x,y,z);

    let [json,arr,num] = [{name:'zhiweili'},[1,2],3];
    console.log(json,arr,num);
```

> 1 2.1 undefined 1 2.1 2.2 { name: 'zhiweili' } [ 1, 2 ] 3

#### 省略赋值

```javascript
let [, , x] = [1, 2, 3];
console.log(x);
```

####  解构对象

对象也可以被解构

```javascript
var obj = {name:'zhiweili',age:8};
//对象里的name属性的值会交给name这个变量，age的值会交给age这个变量
var {name,age} = obj;
//对象里的name属性的值会交给myname这个变量，age的值会交给myage这个变量
let {name: myname, age: myage} = obj;
console.log(name,age,myname,myage);
```

####  默认值

在赋值和传参的时候可以使用默认值

```javascript
let [a = "a", b = "b", c =new Error('C必须指定')] = [1, , 3];
console.log(a, b, c);

function ajax (options) {
    var method = options.method || "get";
    var data = options.data || {};
    //.....
}
function ajax ({method = "get", data}) {
    console.log(arguments);
}
ajax({
    method: "post",
    data: {"name": "zhiweili"}
});
```

### 字符串

#### 模板字符串

模板字符串用反引号(数字1左边的那个键)包含，其中的变量用${}括起来

```javascript
var name = 'zhiweili',age = 8;
let desc = `${name} is ${age} old!`;
console.log(desc);

//所有模板字符串的空格和换行，都是被保留的
var str = `<ul>
<li>a</li>
<li>b</li>
</ul>`;
console.log(str);
```

> 其中的变量会用变量的值替换掉

```javascript
function replace(desc){
  return desc.replace(/\$\{([^}]+)\}/g,function(matched,key){
    return eval(key);
  });
}
```

#### 带标签的模板字符串

可以在模板字符串的前面添加一个标签，这个标签可以去处理模板字符串 标签其实就是一个函数,函数可以接收两个参数,一个是strings,就是模板字符串里的每个部分的字符 还有一个参数可以使用rest的形式values,这个参数里面是模板字符串里的值

```javascript
var name = 'zhiweili',age = 8;
function desc(strings,...values){
    console.log(strings,values);
}
desc`${name} is ${age} old!`;
```

#### 字符串新方法

- includes()：返回布尔值，表示是否找到了参数字符串。
- startsWith()：返回布尔值，表示参数字符串是否在源字符串的头部。
- endsWith()：返回布尔值，表示参数字符串是否在源字符串的尾部。

```javascript
var s = 'zhiweili';
s.startsWith('z') // true
s.endsWith('x') // true
s.includes('p') // true
```

第二个参数，表示开始搜索的位置

```javascript
var s = 'zhiweili';
console.log(s.startsWith('p',2)); // true
console.log(s.endsWith('f',2)); // true
console.log(s.includes('f',2)); // false
```

> endsWith的行为与其他两个方法有所不同。它针对前n个字符，而其他两个方法针对从第n个位置直到字符串结束

#### repeat

repeat方法返回一个新字符串，表示将原字符串重复n次。

```javascript
'x'.repeat(3);
'x'.repeat(0);
```

### 函数

#### 默认参数

可以给定义的函数接收的参数设置默认的值 在执行这个函数的时候，如果不指定函数的参数的值，就会使用参数的这些默认的值

```javascript
function ajax(url,method='GET',dataType="json"){
  console.log(url);
  console.log(method);
  console.log(dataType);
}
```

#### 展开操作符 

把...放在数组前面可以把一个数组进行展开,可以把一个数组直接传入一个函数而不需要使用apply

```javascript
//传入参数
let print = function(a,b,c){
    console.log(a,b,c);
}
print([1,2,3]);
print(...[1,2,3]);

// 可以替代apply
var m1 = Math.max.apply(null, [8, 9, 4, 1]);
var m2 = Math.max(...[8, 9, 4, 1]);

// 可以替代concat
var arr1 = [1, 3];
var arr2 = [3, 5];
var arr3 = arr1.concat(arr2);
var arr4 = [...arr1, ...arr2];
console.log(arr3,arr4);

//类数组的转数组
function max(a,b,c) {
    console.log(Math.max(...arguments));
}
max(1, 3, 4);
```

#### 剩余操作符

剩余操作符可以把其余的参数的值都放到一个叫b的数组里面

```javascript
let rest = function(a,...rest){
    console.log(a,rest);
}
rest(1,2,3);
```

####  解构参数

```javascript
let destruct = function({name,age}){
    console.log(name,age);
}
destruct({name:'zhiweili',age:6});
```

#### 函数的名字

ECMAScript 6 给函数添加了一个name属性

```javascript
var desc = function descname(){}
console.log(desc.name);
```

####  箭头函数

箭头函数简化了函数的的定义方式，一般以 "=>" 操作符左边为输入的参数，而右边则是进行的操作以及返回的值inputs=>output

```javascript
[1,2,3].forEach(val => console.log(val)););
```

> 输入参数如果多于一个要用()包起来，函数体如果有多条语句需要用{}包起来

箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。 正是因为它没有this，从而避免了this指向的问题。

```javascript 
var person = {
    name:'zhiweili',
    getName:function(){
-        setTimeout(function(){console.log(this);},1000); //在浏览器执行的话this指向window
+        setTimeout(() => console.log(this),1000);//在浏览器执行的话this指向person
    }
}
person.getName();
```

####  数组的新方法

##### from

将一个数组或者类数组变成数组,会复制一份

```javascript
let newArr = Array.from(oldArr);
```

#####  Array.of

of是为了将一组数值,转换为数组

```javascript
console.log(Array(3), Array(3).length);
console.log(Array.of(3), Array.of(3).length);
```

##### copyWithin

Array.prototype.copyWithin(target, start = 0, end = this.length) 覆盖目标的下标 开始的下标 结束的后一个的下标

```javascript
[1, 2, 3, 4, 5].copyWithin(0, 1, 2);
```

> 结果：[2, 2, 3, 4, 5]

##### find

查到对应的元素和索引

```javascript
let arr = [1, 2 ,3, 3, 4, 5];
let find = arr.find((item, index, arr) => {
    return item === 3;
});
let findIndex = arr.findIndex((item, index, arr) => {
    return item === 3;
});

console.log(find, findIndex);
```

##### fill

就是填充数组的意思 会更改原数组 Array.prototype.fill(value, start, end = this.length);

```javascript
 let arr = [1, 2, 3, 4, 5, 6];
 arr.fill('a', 1, 2);
 console.log(arr);
 ```

 ##### map

 ##### reduce

 ##### filter

 ##### forEach

 ### 对象

 #### 对象字面量

 如果你想在对象里添加跟变量名一样的属性，并且属性的值就是变量表示的值就可以直接在对象里加上这些属性

```javascript
let name = 'zhiweili';
let age = 8;
let getName = function(){
    console.log(this.name);
}
let person = {
    name,
    age,
    getName
}
person.getName();
```

#### Object.is

对比两个值是否相等

```javascript
console.log(Object.is(NaN,NaN));
```

#### Object.assign

把多个对象的属性复制到一个对象中,第一个参数是复制的对象,从第二个参数开始往后,都是复制的源对象

```javascript
var nameObj = {name:'zhiweili'};
var ageObj = {age:8};
var obj = {};
Object.assign(obj,nameObj,ageObj);
console.log(obj);

//克隆对象
function clone (obj) {
  return Object.assign({}, obj);
}
```

#### Object.setPrototypeOf

将一个指定的对象的原型设置为另一个对象或者null

```javascript
var obj1  = {name:'zhiweili1'};
var obj2 =  {name:'zhiweili2'};
var obj = {};
Object.setPrototypeOf(obj,obj1);
console.log(obj.name);
console.log(Object.getPrototypeOf(obj));
Object.setPrototypeOf(obj,obj2);
console.log(obj.name);
console.log(Object.getPrototypeOf(obj));
```

#### proto

直接在对象表达式中设置prototype

```javascript
var obj1  = {name:'zhiweili1'};
var obj3 = {
    __proto__:obj1
}
console.log(obj3.name);
console.log(Object.getPrototypeOf(obj3));
```

#### super
通过super可以调用prototype上的属性或方法

```javascript
let person ={
    eat(){
        return 'milk';
    }
}
let student = {
    __proto__:person,
    eat(){
        return super.eat()+' bread'
    }
}
console.log(student.eat());
```

### 类

#### class

使用class这个关键词定义一个类,基于这个类创建实例以后会自动执行constructor方法,此方法可以用来初始化

```javascript
class Person {
    constructor(name){
        this.name = name;
    }
    getName(){
        console.log(this.name);
    }
}
let person = new Person('zhiweili');
person.getName();
```

#### get与set

getter可以用来得获取属性，setter可以去设置属性

```javascript
class Person {
    constructor(){
        this.hobbies = [];
    }
    set hobby(hobby){
        this.hobbies.push(hobby);
    }
    get hobby(){
        return this.hobbies;
    }
}
let person = new Person();
person.hobby = 'basketball';
person.hobby = 'football';
console.log(person.hobby);
```
#### 静态方法-static

在类里面添加静态的方法可以使用static这个关键词，静态方法就是不需要实例化类就能使用的方法

```javascript
class Person {
   static add(a,b){
       return a+b;
   }
}
console.log(Person.add(1,2));
```

#### 继承extends

一个类可以去继承其它的类里的东西

```javascript
class Person {
   constructor(name){
     this.name = name;
   }
}
class Teacher extends Person{
    constructor(name,age){
        super(name);
        this.age = age;
    }
}
var teacher = new Teacher('zhiweili',8);
console.log(teacher.name,teacher.age)
```

### 生成器(Generator)与迭代器(Iterator)

Generator是一个特殊的函数，执行它会返回一个Iterator对象。 通过遍历迭代器， Generator函数运行后会返回一个遍历器对象，而不是普通函数的返回值。

#### Iterators模拟

迭代器有一个next方法，每次执行的时候会返回一个对象 对象里面有两个属性，一个是value表示返回的值，还有就是布尔值done,表示是否迭代完成

```javascript
function buy(books) {
    let i = 0;
    return {
        next(){
            let done = i == books.length;
            let value = !done ? books[i++] : undefined;
            return {
                value: value,
                done: done
            }
        }
    }
}

let iterators = buy(['js', 'html']);
var curr;
do {
    curr = iterators.next();
    console.log(curr);
} while (!curr.done);
```

#### Generators

生成器用于创建迭代器

```javascript
function* buy(books){
    for(var i=0;i<books.length;i++){
        yield books[i];
    }
}
let buying = buy(['js','html']);
var curr;
do {
    curr = buying.next();
    console.log(curr);
} while (!curr.done);
```

### 集合

#### Set

一个Set是一堆东西的集合,Set有点像数组,不过跟数组不一样的是，Set里面不能有重复的内容

```javascript
var books = new Set();
books.add('js');
books.add('js');//添加重复元素集合的元素个数不会改变
books.add('html');
books.forEach(function(book){//循环集合
    console.log(book);
})
console.log(books.size);//集合中元数的个数
console.log(books.has('js'));//判断集合中是否有此元素
books.delete('js');//从集合中删除此元素
console.log(books.size);
console.log(books.has('js'));
books.clear();//清空 set
console.log(books.size);
```

#### Map

可以使用 Map 来组织这种名值对的数据

```javascript
var books = new Map();
books.set('js',{name:'js'});//向map中添加元素
books.set('html',{name:'html'});
console.log(books.size);//查看集合中的元素
console.log(books.get('js'));//通过key获取值
books.delete('js');//执照key删除元素
console.log(books.has('js'));//判断map中有没有key
books.forEach((value, key) => { //forEach可以迭代map
    console.log( key + ' = ' + value);
});
books.clear();//清空map
```

### 模块

在浏览器中使用模块需要借助 导出

```javascript
export var name = 'zhiweili';
export var age = 8;
```

导入

```javascript
//import {name,age} from './school.js';
import * as school from './school.js';
console.log(school.name,school.age);
```

在页面中引用

```javascript
<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
<script type="module" src="index.js"></script>
```

####  重命名

导出时重命名

```javascript
function say(){
    console.log('say');
}
export {say as say2};
```

导入时重命名

```javascript
import {say2 as say3} from './school.js';
```

#### 默认导出

每个模块都可以有一个默认要导出的东西 导出

```javascript
export default function say(){
    console.log('say');
}
```
导入

```javascript
import say from './school.js';
```
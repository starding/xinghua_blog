---
title: javascript 犀牛书（第六版）
og_title: javascript 犀牛书（第六版）
date: 2016-01-19 13:01:58
category: 作为工程师
---
记录阅读《javascript 权威指南》中，个人认为比较核心或是比较容易混乱的地方。
同时会加入在阅读过程中参阅的一些优秀的扩展文章。

**更新记录**
> 2016-01-18 更新到 try-except-finally 语句
> 2016-04-07 文章改名。将文章结构分为核心和模糊点

**我认为书中的核心部分**
> 1. 语言核心部分
> 2. 变量类型
> 3. strict 模式
> 4. 词法作用域、函数，闭包、this、call、apply、bind
> 5. 原型系统

## 核心部分
### 语言核心
#### 变量及变量类型
javascript 中的变量本身没有类型，给一个变量赋值时，其实是这个变量引用了这个值。这根 python 中是一致的。 

他们都区别于 C 语言等固定类型的变量，在 C 语言中，当声明一个变量时，需要指定它的类型，这个变量其实就是一个固定大小的空间，直接用于存放变量的值。数组也是一样，一个数组只能存储同样类型的值，C 语言需要根据类型决定如何分配内存大小，比如 int 是 4 个字节。

而 javascript 和 Python 中，无需声明变量类型。变量只是对数据的引用，而类型是数据本身决定的，这个变量引用了什么类型，这个变量就是什么类型，javascript 中的数据类型分为两类（貌似是借鉴了 java）:

#### 原始类型
* 原始类型 primitive type（5 种）
  * 数字
  * 字符串
  * 布尔值
  * null
  * undefined

**注意：**原始类型不是对象，但你可能会见过 APrimitiveType.method 的调用形式，这是因为数字，字符串，布尔值存在「包装对象」这个概念。

#### 对象类型
* 对象类型（object）（除了 5 种原始类型的数据类型，都是对象类型）
  * 普通对象（名值对对象）
  * 数组
  * 函数 - javascript 可以将 function 对象当做普通对象对待

#### 严格模式：use strict
如果你了解一些 javascript 的历史，就会清楚 javascript 是 Brendan Eich 花了十天时间设计出来的，主要参考了 Scheme，Self，C，Java 等语言。虽然有现成的语言可以借鉴，但是在如此短的时间内，想要完成一个非常严格的设计是不太现实的。因此 Javascript 本身在设计之初就带有一些隐藏的缺陷。再加上浏览器大战时的迅速推广，这些缺陷没有足够的时间得到修正，便开始成为标准。所以使得这些缺陷一直保留了下来。

关于这些缺陷，可以简单参考阮一峰的文章：[javascript 的十个设计缺陷](https://www.ruanyifeng.com/blog/2011/06/10_design_defects_in_javascript.html)，以及书籍：《javascript: the good parts》

use strict 其实是 ECMAscript 5 中加入的指令，非常接近于语句，但又有所区分：不包含关键字，只是个字符串字面量，而且只能出现在特定的位置。

use strict 的目的是使其后的代码使用 js 的「严格模式」执行，它是 js 的一个子集，修正了一些语言的重要缺陷，并提供健壮的查错功能和增强的安全机制。严格模式与非严格模式有一些重要的区别。

* 严格模式中不能使用 with 语句
* 严格模式中变量都要先 var 声明（非严格模式不声明使用会给全局对象添加一个新属性）
* 严格模式中，函数调用时 this 值为 undefined，而不是全局对象。

## 模糊点
### 包装对象
原始类型不是对象，但你可能会见过 APrimitiveType.method 的调用形式，这是因为数字，字符串，布尔值存在「包装对象」这个概念。也就是说，档使用上面的。运算时，javascript 会自动调用 new String(s) | new Number(s) | new Boolean(s) 的方式将这三种基本类型临时转换为对象，并继承相应类型的方法。

```javascript
//一个经典的例子
var s = "test";
s.len = 4;  //因为点运算，这里会临时建立一个 string 对象，但是随即又销毁了
var t = s.len; //这里也会创建一个 string 对象，但是已经不是上面那个了，故 undefined
```

* null 和 undefined 没有包装对象
* 可以使用构造函数显式地构造包装对象。

### 变量声明提前和变量可配置性
* 声明提前
  * 指的是 var 声明变量时，会提前到其所在作用域的顶部
* 可配置性
  * 全局变量是全局对象的属性，但局部变量没有此规定
  * var 声明的全局变量是不可配置的，无法通过 delete 运算符删除

### eval 运算符
eval() 实际上是个函数，但它早已被当成运算符看待。ECMAscript 会尽量约束 eval() 的表现，使其看起来像个表达式。

这是因为如果将 eval 当做普通的函数对待，不做任何限制的话，会有很多缺点。一是因为 eval() 会动态的执行代码段，破坏原来的代码结构，解释器难以优化。二是因为如果 eval 是普通的函数，那么它就可以被赋值给其他的变量名，比如 g = eval, 这样导致任何使用 g 的地方，也会产生上述问题，导致 javascript 的代码优化变得困难，性能堪忧。

综上所述，如果 eval 的表现行为像一个运算符的话，就不会有上述这么严重的问题。所以在 ECMAscript 语法中不断的约束它，让其表现更接近为运算符。

### 标签语句
主要用于配合 continue 和 break 使用，来实现更灵活一些的跳转。

### throw/try/catch/finally 语句
之前写过一篇[那些年我在 python 中扑过的街]()，在其中提到过 python 中，try/except/finally 中容易进的坑。

这个坑在 javascript 中也是同样的，就像那篇文章里说的那样，因为 finally 的设计目的就是无论如何都会执行，那么一旦 finally 中出现了跳转语句，其他语句中的返回内容就会被忽略。

```javascript
function finallyTest(){
    var x = 'catch', y = 'except', z = 'finally';
    try {
        k = x + y;
        a = b;
        console.log("here is try------------");
    } catch(e) {
        console.log("here is catch----------");
        return x;
    } finally{
        console.log("here is finally---------");
        return z;
    }
}
//执行
finallyTest()
//返回值
here is catch----------
here is finally---------
'finally'
```

### 函数名
```js
// 直接声明函数是具有函数名的
function foo(){}
foo.name  // 函数名为 foo
var bar = foo
bar.name  // 将函数赋值给一个变量，只是让变量指向了函数结构，并不会改变函数结构本身
// 函数表达式生成的是匿名函数
var foo = function(){}
foo.name // "" (空字符串)
// 函数名称只存在于函数自身结构内，被传递时不会改变这个结构
function foo(){}
var o ={
    a: 'a',
    bar: foo
};
o.bar.name // foo
```

### this 引用丢失
```js
// 函数被多次传递时可能发生
function foo(){
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo;
var a="oops, global";
bar(); //"oops, global"
// 回调函数中可能发生（异步）
// 代码同上
setTimeout(obj.foo, 100); // "oops, global"
```
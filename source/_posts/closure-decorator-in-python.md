---
title: Python 中的闭包和装饰器
og_title: Python 中的闭包和装饰器
date: 2016-01-06 22:11:43
category: 作为工程师
---

第一版创建于：2016-01-06

## 闭包
首先看维基百科中的定义：
> In programming languages, closures (also lexical closures or function closures) are a technique for implementing lexically scoped name binding in languages with first-class functions. Operationally, a closure is a record storing a function together with an environment: a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or storage location to which the name was bound when the closure was created. A closure—unlike a plain function—allows the function to access those captured variables through the closure’s reference to them, even when the function is invoked outside their scope

## 然后考察定义：
上面这段话中有几个关键点：

* 闭包是一项技术（technique）
* 一个闭包是一个函数和它所在的环境构成的记录
* 闭包，也就是上面说的记录，创建时，形成了函数中用到的自由变量（在封闭的词法作用域中定义，但在函数本地使用的变量）与其值的绑定关系。
* 闭包不同于普通的函数，它允许函数使用已经捕获（绑定）的变量，即使变量的定义在其作用域之外。
直接考察定义，再加上对「闭包」一词的直觉，只能产生一种模糊的概念：这货大致是一个封闭的结构，它包括一个函数以及在函数外定义的变量。但是更深入的理解就还得看实际中的代码了。

一个 python 闭包的例子：
```python
# 创建闭包的函数
def counter(start_at=0):
	count = [start_at]
	def incr():
		count[0] += 1
		return count[0]
	return incr
# 将闭包赋值给另一个变量
counter1 = counter()
# 再次赋值给一个变量
counter2 = counter(5)
```

在上面的代码中，counter 是一个创建闭包的函数，然后下面两句分别赋值给了两个不同的变量。这实际上产生了两个闭包函数，counter1 和 counter2，他们都包括一个函数本身，以及在函数外关联的一个作用域。

上面的两个闭包非常像实例化的两个对象，它们附加的那个函数之外的作用域互相独立。彼此之间互不影响。

## 装饰器
首先解释字面意思
装饰器这个名词，字面意思就是「装饰其他特定东西用的工具」。这个词里暗含着一层意思，就是被装饰的东西才是核心，而装饰用的工具，只不过是起到点缀作用，增加点额外的东西罢了。

以一个隐喻来做比：在圣诞节的时候，我们会拿一些小挂件去装饰圣诞树，那么这个时候的小挂件就是一个一个的小装饰器，被装饰的核心是圣诞树。这也暗含着，核心是圣诞树，这些小挂件只不过是给圣诞树增加一些其他有趣的特性罢了。

回到 python 中也是类似的，在 python 中，并不是像存在函数，字典，列表这些 python 元素一样，真的有一种对象类型叫装饰器。而是说，python 中可以构造这样一种可调用对象（一般是函数或类来构造），它可以用于装饰别的对象，我们就把具有这样装饰功能的对象叫做装饰器。其实质不过是起到装饰作用的一些可调用对象。

那么这样就有两个问题：

装饰什么呢？
又是如何实现装饰功能的呢？
还是通过代码来看，一个 python 装饰器的例子：

```python
import time
from functools import wraps
# 被装饰的主体函数
def foo():
	# 做一些我们想要做的功能
	# ...
	# ...
	# 最后打印一下
	print 'in foo()'
# 装饰器	
def timeit(func):
	'''
	Decorator that reports the execution time.
   '''
   @wraps(func)
	def wrapper(*args, **kwargs):
		start  = time.clock()
		result = func(*args, **kwargs)
		end = time.clock()
		print 'used: {0}'.format(end-start)
		return result
	return wrapper
```

首先要明白我们的程序主体是什么，我们想要用 foo 函数来完成一些特定的功能，最终再打印一下，所以我们的主体函数是 foo。

同时，我们还想知道完成这件事所用的时间，那这正好可以通过装饰器来完成。于是我们构造了一个函数 timeit 作为装饰器，来装饰 foo 函数。

需要再次强调的一点是，装饰器 timeit 是用来装饰 foo 函数的，它只是给 foo 函数增加可以输出执行时间的特性，并不改变 foo 函数原来想要完成的功能。对 foo 函数原来所要做的事情，没有任何影响。

不过如果你留心一下，就会发现上面这个装饰器的实现中，用到了闭包这一技术（只是不那么明显，因为并没有带上一个额外的作用域，加上一个也是没问题的）。更准确的说，闭包和装饰器都是基于 python 中的可调用对象可以传递这一事实。

## 装饰器模式
那么装饰器是怎么来的呢？其实这一概念来源于设计模式中的装饰器模式：在不改变核心调用对象的情况下，给它添加一些有趣的也可能很有用的特性。这正是“装饰”的意义所在。

这不仅既不用改动原来的核心对象，又达到了我们想要的目的，实在是高明。

2016.01 于北京回龙观
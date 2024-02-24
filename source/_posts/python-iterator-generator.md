---
title: python 中的装饰器和迭代器
date: 2016-01-15 09:01:40
category: 作为工程师
---
简单整理一下吧，毕竟输出才是检验自己水平更好的方法。而且感觉网上很多文章都不是很靠谱，很明显甚至连官方文档都没有看，就只根据经验就直接写文章来总结了，很容易误导人。

## Iterator Types
迭代器类型

### 先看 python2.7 官方文档中关于迭代的部分：
在阅读之前，需要明确几个词的含义：

- 「iterator」迭代器
- 「iteration」迭代
- 「iterable」可迭代的
- 「container」容器对象，也就是可以存放其他对象的对象，比如列表，字典等

> New in version 2.2.
> Python supports a concept of iteration over containers. This is implemented using two distinct methods; these are used to allow user-defined classes to support iteration. Sequences, described below in more detail, always support the iteration methods.
> python 提供一种基于容器对象（container）的迭代概念。并且使用了两个特的方法来实现这一概念；这些方法允许自定义的类支持迭代。符合下面详细描述的序列（sequences），都将支持迭代方法。

> One method needs to be defined for container objects to provide iteration support:
> 第一个在容器对象中定义的方法用于支持迭代（iteration）
> `container.__iter__()`
> Return an iterator object. The object is required to support the iterator protocol described below. If a container supports different types of iteration, additional methods can be provided to specifically request iterators for those iteration types. (An example of an object supporting multiple forms of iteration would be a tree structure which supports both breadth-first and depth-first traversal.) This method corresponds to the tp_iter slot of the type structure for Python objects in the Python/C API.
> 返回一个迭代器对象（iterator object）。这个对象需要支持下面描述的迭代器协议（iterator protocal）。如果一个容器对象需要支持不同类型的迭代方式，也可以往容器对象中添加更多的方法来支持那些迭代方式。（一个对象支持多种跌打方式的例子是同时支持深度优先遍历和广度优先遍历的树结构）

> The iterator objects themselves are required to support the following two methods, which together form the iterator protocol:
>迭代器本身也需要支持下面的两个方法，『这两个方法一起构成了迭代器协议』

> `iterator.__iter__()`
> Return the iterator object itself. This is required to allow both containers and iterators to be used with the for and in statements. This method corresponds to the tp_iter slot of the type structure for Python objects in the Python/C API.
> 返回迭代器对象自身。如果在 for 语句以及 in 语句中，使用容器对象和迭代器对象的话，那么它们都需要拥有此方法。


> `iterator.next()`
> Return the next item from the container. If there are no further items, raise the StopIteration exception. This method corresponds to the tp_iternext slot of the type structure for Python objects in the Python/C API.
> 返回容器中的下一个元素。如果没有下一个元素，则产生一个 StopIteration 异常。
> Python defines several iterator objects to support iteration over general and specific sequence types, dictionaries, and other more specialized forms. The specific types are not important beyond their implementation of the iterator protocol.
> python 基于一些或通用或特定的序列类型，字典或其他更特殊的类型，定义了一系列的迭代器对象来支持迭代。这些对象的核心内容就是其中的迭代器协议。
> The intention of the protocol is that once an iterator’s next() method raises StopIteration, it will continue to do so on subsequent calls. Implementations that do not obey this property are deemed broken. (This constraint was added in Python 2.3; in Python 2.2, various iterators are broken according to this rule.)
> 这个协议的目的是，一旦一个迭代器的 next() 方法产生了一个 StopIteration 异常，后续再度调用时，就会一直保持产生一个 StopIteration 的状态。如果不按照这个特性实现，就被认为是有问题的（这个约束是在 python2.3 中添加的；在 python2.2 中，有很多迭代器不遵守这个规则）

### 概念辨析
上面大致就是 python2.7 中关于迭代的核心内容，整理一下就是：

对于容器对象（container）来说，要支持迭代的话，需要在容器内部实现一个__iter__() 方法。这个方法返回一个「迭代器对象」（iterator）。如果一个容器实现了这个方法，那么我们称这个容器是「可迭代的」（iterable）。

对于迭代器来说，它也需要一个__iter__() 方法，用于返回这个迭代器自身。同时需要一个 next() 方法，来返回下一个元素。迭代器本身当然是「可迭代的」。迭代器的这两个方法，合在一起，叫做「迭代器协议」。

在 for，in 语句中，无论是使用容器对象（如列表），还是迭代器对象，它们内部都需要支持__iter__() 方法。对于容器对象来说这个方法会返回一个迭代器，对于迭代器对象来说，这个方法会返回自身。然后用于迭代，换句话说，即使是容器对象，也是先转换为迭代器对象再进行迭代的。这在 python 的文档中也有说明：

### The for statement
> The for statement is used to iterate over the elements of a sequence (such as a string, tuple or list) or other iterable object:
> 
> ```py 
> for_stmt ::=  "for" target_list "in" expression_list ":" suite ["else" ":" suite]
> ```
> The expression list is evaluated once; it should yield an iterable object. An iterator is created for the result of the expression_list. The suite is then executed once for each item provided by the iterator, in the order of ascending indices. Each item in turn is assigned to the target list using the standard rules for assignments, and then the suite is executed. When the items are exhausted (which is immediately when the sequence is empty), the suite in the else clause, if present, is executed, and the loop terminates.

## Generator Types
生成器类型

### 官方文档的定义：
> Python’s generators provide a convenient way to implement the iterator protocol. If a container object’s `__iter__()` method is implemented as a generator, it will automatically return an iterator object (technically, a generator object) supplying the `__iter__()` and next() methods. More information about generators can be found in the documentation for the yield expression.
> python 的生成器提供了应用迭代器协议的便捷方式。如果一个容器对象的`__iter__()` 方法被用于生成器，它会自动地返回一个迭代器对象（技术上讲，是一个生成器对象），并且提供_`_iter__()` 和 `next()` 方法。

### 概念辨析
上述内容的意思是，生成器对象不过是一种应用迭代器协议的快捷方式。平时我们需要一个自定义的迭代器
时，需要手动的去实现需要的`__iter__()` 和 `next()` 方法，但是使用生成器对象的话，可以自动返回一个支持迭代协议的迭代器。而创建一个生成器，只需要在函数中使用 yield 表达式就可以了，这样会创建一个生成器函数，当它被调用时，会返回一个迭代器（通常被叫做生成器），它会控制生成器函数的执行。

> The yield expression is only used when defining a generator function, and can only be used in the body of a function definition. Using a yield expression in a function definition is sufficient to cause that definition to create a generator function instead of a normal function.
> 
> When a generator function is called, it returns an iterator known as a generator. That generator then controls the execution of a generator function. The execution starts when one of the generator’s methods is called. At that time, the execution proceeds to the first yield expression, where it is suspended again, returning the value of expression_list to generator’s caller. By suspended we mean that all local state is retained, including the current bindings of local variables, the instruction pointer, and the internal evaluation stack. When the execution is resumed by calling one of the generator’s methods, the function can proceed exactly as if the yield expression was just another external call. The value of the yield expression after resuming depends on the method which resumed the execution.
>
> All of this makes generator functions quite similar to coroutines; they yield multiple times, they have more than one entry point and their execution can be suspended. The only difference is that a generator function cannot control where should the execution continue after it yields; the control is always transferred to the generator’s caller
---
title: 什么是 Javascript Event Loop
og_title: 什么是 Javascript Event Loop
date: 2016-04-26 15:47:41
tags:
---
本文为翻译文章，[原文](https://altitudelabs.com/blog/what-is-the-javascript-event-loop/)[^1](2024 年，作者的博客已经无法再访问了，好在 Medium 上有一个备份：https://medium.com/@fay_jai/what-is-the-javascript-event-loop-98707ed20a90)

## 引言
如果你像我一样热爱 JavaScript。没错，它不是完美的语言，但是这个世界上哪有「完美的语言」这种存在？所以尽管 Javascript 有这样或那样的缺陷，我仍然喜欢 web 编程以及 JavaScript 给予我的编写连接这个世界的应用的能力。

但是 JavaScript 是有深度的——它有一个复杂的内部机制，你需要花费一定的时间去理解。其中一个有深度的地方就是 Event Loop。当然，即便是在对 JavaScript 的 Event Loop 没有精确理解的情况下，也能在很长时间内，正常使用 javascript 进行编程。然而，我希望本篇博客能带你走入 Event Loop 的世界，让你意识到这玩意儿并不是那么难以理解。

## 浏览器中的 JavaScript
当我们思考 JavaScript 的时候，我们通常会默认一个语境前提——web 浏览器——这使得大多数人都是在客户端编写 JavaScript 代码。事实上，意识到运行任何 web 应用实际上包含一些列技术，像 JavasCript 引擎（如 Chrome 的 V8 引擎），一堆 Web API（如 DOM），还有时间轮询（Event Loop）和事件队列（Event Queue）。

看到上面这些内容之后，你可能会想：『艹，这下看起来更复杂了…』——或许的确是这样——但是你很快就会看到，上面这些技术的核心思想真的没有那么复杂，甚至你可能发现，实现它们会非常容易。

在深入研究 event loop 之前，我们需要对 JavaScript 引擎和它的工作原理有一些基本的理解。

## JavaScript 引擎
目前不同的 JavaScript 引擎实现有好几种，但是当前最流行的实现版本是 Google Chrome 的 V8 引擎（这个引擎不受限于浏览器端，在服务器端的 Nodejs 使用的也是它）。说了这么多，那 JavaScript 引擎到底是干啥的？其实很简单——它的工作就是遍历 Web 应用中的每一行 JavaScript 代码，并且逐个执行（process one at a time）。你没有看错——就是逐个执行，这意味着 JavaScript 是单线程的。这一特点的主要影响是，如果你执行一行需要非常非常长的时间才能返回结果的代码，这之后的代码都会被阻塞掉。我们当然不想写出这么阻塞的代码——特别是在浏览器中。想象一下你正在查看一个 Web 网页并且垫底了某个按钮…它就这么耗在了那里。你试着点击其他的按钮，然而并没有什么卵用，啥作用都没起。这种蛋疼的局面只能归罪于点击按钮的时候触发了某些代码的执行（假设没有 bug 的话），但是这些代码阻塞在了那里。

另一方面，JavaScript 引擎如何知道一次只执行一行代码的？它使用了一个调用栈（call stack）。你可以把调用栈想象成一个叠罗汉活动——第一个叠罗汉的只能最后一个下来，对应的是最后一个叠罗汉的第一个下来（原文用乘电梯作比喻，个人认为不是很形象，改成了叠罗汉）。

下面来看一个例子：

```javascript
/* Within main.js */
var firstFunction = function () {  
  console.log("I'm first!");
};
var secondFunction = function () {  
  firstFunction();
  console.log("I'm second!");
};
secondFunction();
/* Results:
 * => I'm first!
 * => I'm second!
 */
```

下面是执行代码中，调用栈中发生的一系列事件：

1. Main.js 首先被执行
![初始状态](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.1evqqj6whq2o.webp)

2. secondFunction 被调用
#[secondFunction 被调用之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.157h49iavv34.webp)

3. 调用 secondFunction 的时候，内部函数 firstFunction 也被调用：
![firstFunction 被调用之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.6v22n4e2e400.webp)

4. 执行 firstFunction 的时候，会输出”I’m first!”,并且由于在 firstFunction 中，没有其他代码要执行，整个 firstFunction 的执行到此结束，被移出调用栈：
![当 firstFunction 返回之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.2ehg5fnit9jw.webp)

5. firstFunction 执行结束返回之后，secondFunction 继续执行，输出”I’m second!”。一旦输出完毕之后，secondFunction 函数中，也没有其他代码要执行，整个 secondFunction 函数执行完毕，被移出调用栈：
![当 secondFunction 返回之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.4i4gxzrtf400.webp)

6. 最后，由于 main.js 中没有其他代码要执行，main.js 也被移出调用栈：
![当 main.js 返回之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.4ga09okx9j40.webp)

## OK，但是这根 Event Loop 有毛线关系？
现在你已经明白 JavaScript 引擎中的调用栈是如何工作的了，让我们回到之前的代码阻塞的思路上来。当然，你已经意识到应该避免出现这些阻塞，但是该怎么做？幸运的是，JavaScript 提供了一种机制，它基于异步回调函数（asynchronous callback function）的方式来实现。这个概念看起来有些吓人，不过不用担心——所谓异步函数和你在 JavaScript 中使用的普通函数没有什么两样，只不过是加了一些它会在之后执行（而不是立即）执行的手段。如果你使用过 Javascript 的 setTimeout 函数，你其实已经使用过异步回调函数的概念了。下面来看一个例子：

```javascript
/* Within main.js */
var firstFunction = function () {  
 console.log("I'm first!");
};
var secondFunction = function () {  
 setTimeout(firstFunction, 5000);
 console.log("I'm second!");
};
secondFunction();
/* Results:
 * => I'm second!
 * (And 5 seconds later)
 * => I'm first!
 */
```

下面是调用栈中的一系列活动（我们省去了前面的函数加入调用栈的过程，直接来看 setTimeout 函数）：

1. 当 secondFunction 被推入调用栈之后，setTimeout 函数被调用，而且也被压入调用栈中：
![在 setTimeout 函数执行之前](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.59jdii43mas0.webp)

当 setTimeout 函数被执行的时候，发生了一些比较特殊的事——浏览器把 setTimeout 的回调函数（在本例中是 firstFunction）存入一个 Event Table 中。可以将 Event Table 想象成一个电话注册本：调用栈会告诉 event table 注册一些特定的函数，并且在指定事件发生时会调用他们。当这些指定事件发生时，event table 仅仅是简单地把要调用的函数移入 Event Queue 中去。event queue 的美妙之处在于它提供了一个简单等待区域，函数在此区域内等待被移入调用栈进行调用。
你或许会问：『究竟什么情况下，event queue 中的函数才会被移入调用栈中？』。实际上，JavaScript 遵从一个简单的法则：存在一个监控进程不断检查调用栈是否为空，当调用栈为空的时候，检查事件队列（event queue）中是否有待调用的函数。如果事件队列中存在待调用的函数，队列头部的函数被移入调用栈执行。如果事件队列为空，监控进程就保持轮询状态。

瞧，我刚才描述的内容就是臭名昭著的 Event Loop（事件轮询）了！

2. 现在回到我们之前的 setTimeout 的例子。执行 setTimeout 函数的过程中，引擎将它的回调函数（在本例中为 firstFunction）添加到 event table 中，同时注册触发事件为 5 秒延迟。
![当 setTimeout 函数被执行之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.5nzgn34nn8k0.webp)

3. 下面是另一个让你觉得「啊哈，是这样！」的时刻——注意到一旦回调函数被移入 event table 之后，没有代码在阻塞了！浏览器在执行后面的任何代码之前，并不会在那里傻等 5 秒了——它直接执行 secondFunction 函数中 setTimeout 函数后面的代码，在这里是 console.log 语句。
![secondFunction 执行完毕之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.4ahr7wpq7j80.webp)

4. 在后台，event table 一直监控是否有指定的事件发生，如果有将触发把对应的函数移入事件队列（event queque）的动作。在上面的例子中，secondFunction 到这里已经执行完毕，于此同时 main.js 到这里也执行完成了。
![当 main.js 执行完成之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.2chkily7e1wk.webp)

5. 大约算来，在回调函数放入 event table 后 5 秒钟，event table 会把 firstFunction 移入事件队列中。
![main.js 执行结束后约 5 秒钟](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.27kc1s3tmn6s.webp)

6. 由于事件循环不断监视调用堆栈是否为空，因此它现在注意到调用堆栈确实为空，并调用 firstFunction 创建一个新的调用堆栈。
![新的调用栈](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.27kc1s3tmn6s.webp)

7. 一旦 firstFunction 执行完毕之后，我们会回到调用栈为空的状态，这个时候 event table，event queue 也都为空。
![当 firstFunction 执行完毕之后](https://cdn.jsdelivr.net/gh/starding/picx-images-hosting@master/20240224/image.31bivw5r6yo0.webp)

## 总结
我承认我上面的解释掩盖了 JavaScript 引擎中，以及 event table，event queue 和 event loop 中非常多的实际实现细节。但是，对于大部分人来说，我们仅仅需要对 JavaScript 执行异步函数时发生的事情有一个笼统的认识。我希望上面的解释能帮助你明晰背后的机制，并且满足平常的 web 开发工作。


## 扩展问题
问题：setTimeout 中设置的时间是准确的吗？
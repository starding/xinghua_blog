---
title: 什么是模态框（modal dialog）
og_title: 什么是模态框（modal dialog）
date: 2016-03-19 15:13:33
category: 作为工程师
---
## 问题来源
在学习前端的时候，我遇到一个让人费解的词：「模态框」，它对应一种特殊的弹出式窗口。而且一时之间很难说清楚到底特殊在什么地方，以及这样的弹窗与其他类型窗体的核心区别在哪里。与「对话框」，「面板」这些比较直白的词汇不同，「模态」这个词处处透漏着一种玄乎的感觉，很难直观的感受到它到底是个啥，又想要通过这个词来表达什么，你会不自觉的感到困惑：啥叫模态？

在计算机领域的词汇中，我一直抱有这样一种信念：如果一个词汇显得玄乎，那它背后肯定隐藏着更多的东西，或许是某些隐喻，或许是一些更深层次的设计原理等等。无论如何，都值得去好好调查一番这些词汇背后的含义，可能这些小小的词汇就成了通往一个个饶有趣味的世界的门径。

## 什么是「模态」？
要理解模态框的含义，除了先用眼睛看看这个词汇对应的东西到底是个啥（可以找 bootstrap 的模态框组件体验）之外，最应该干的事情，就是先搞明白「模态」的含义了。

**模态，英文词汇叫 modal**

先看看「模态」的字典解释：模式的，情态的，形式的。

再看看使用这个词汇组合出来的一些让人莫名其妙的专业词汇。

* 模态分析
* 模态矩阵
* 模态逻辑
* 模态框

我们可以猜想一下这个词汇的引入情景，或者说从英文词汇 modal 翻译成汉语的情形：想要表达一种特定状态下的内容，那该怎么翻译比较好呢？我们知道「模型」这个词有「一种事物的固定抽象」的含义，那么它可以表达一种「固定模式的含义」，而「状态」一词，可以表示事物在某种情形下的表现。这两者结合一下，「模态」这个词便呼之欲出了。当然，实际的翻译和词汇创造肯定远远比上面描述的情况复杂，但核心思想是一致的，也即：如何信达雅的表示出一个事物的概念。

经过这样的分析，我们可以说算是对「模态」这个词稍有了解了，它指的是**某种特定的状态**。

这个时候再来看看上面那些专业词汇，就比较容易理解了，也就是说他们都有一种**「研究某些特定状态下的事物」**的意思。

## 什么是「模态框」
有了上面的词汇理解基础，我们可以继续往下说模态框的概念了。从字面意思上来看，它指的应当是「某种特定状态下的窗体」。

当然，仅仅这样来看，可能仍然有些难以理解。这是因为我们缺了另外一些东西造成的，缺少的就是「模态框」这个东西使用的语境，只有加入这些内容才能让我们的理解完整。

那么模态框使用的语境是什么？当然，这个词是在软件领域产生的，我们可以先看看普通的软件使用流程。在使用软件的时候，我们一般都会按照自己的思路一步步操作，比如我们在使用一个购物系统，我们会按照我们对这个软件的固有理解来执行自己脑中的流程：选购商品，加入购物车，下单付款等等。这些流程可以说是我们使用软件时的一种「正常状态」。

「模态框」这种『特定状态下的窗体』正是相对于上面叙述的这种「正常状态」来说的。模态框是出于一种特定状态下的窗体，它会把我们从正常状态中中断出来，将关注点放在这个特定状态的处理上。可以看看模态框的实际表现：当模态框出现的时候，它会屏蔽掉所有其他操作，用户可关注的范围只限于当前的模态框内部，除非你特意去关闭这个模态框，结束这种中断，回到原先正常的流程中去。

上面所描述的就是模态框的核心思想。其实准确地说，模态框是一个 UI 设计领域的概念，维基百科的定义是：

> **model window**
> 
> In user interface design, a modal window is a graphical control element subordinate to an application’s main window which creates a mode where the main window can’t be used. The modal window is a child window that requires users to interact with it before it can return to operating the parent application, thus preventing the workflow on the application main window. Modal windows are often called heavy windows or modal dialogs because the window is often used to display a dialog box.
> 
> Modal windows are commonly used in GUI systems to command user awareness and to display emergency states, although they have been argued to be ineffective for that use. Modal windows are prone to produce mode errors.

当然模态框这种设计理念，暗含着一种强制性的思路。它强制用户的关注点从正常思维流中抽出来，来关注模态框内的内容，有些强制思考的意味。这种设计理念一般用在比较危险的操作的提示上。

但是对模态框的批评也是多种多样，主要是批评这种强制性的设计思路，以及它是否应被更好的方案代替等等，更有些观点宣称模态框是「邪恶的」。关于这些批评，可以参考下面一些资料：

* [Why are modal dialog boxes evil?](https://stackoverflow.com/questions/361493/why-are-modal-dialog-boxes-evil)
* [Never Use a Warning When you Mean Undo](https://alistapart.com/article/neveruseawarning/)

## 「非模态」
有模态的概念，当然也有非模态的概念，关于非模态概念，本文暂时按下不表。
---
title: 问题记录
date: 2016-02-25 09:25:29
category: 作为工程师
---
Meta: 本文章用来记录平时遇到的问题以及相应的解决方案

## 杂项
### linux crontab + django command 定时执行命令
#### 问题场景
在做一个档案管理系统时，有一个需要定时从 DB2 数据库中取出数据，处理之后，存入 mysql 中的任务。这意味着需要在 django 中单独跑一个脚本，这个脚本使用 django 的环境。

#### 解决方案
django 提供了一种用户自定义命令来实现这个功能：

> [Writing custom django-admin commands¶](https://docs.djangoproject.com/en/1.9/howto/custom-management-commands/)

而 linux 则提供了定时执行任务方面的好用工具：

> [linux crontab](https://linuxtools-rst.readthedocs.io/zh-cn/latest/tool/crontab.html)

### 往 mysql 中写大量数据时，出现错误：2006‘mysql server has gone away’
#### 问题场景
在做一个档案管理系统时，使用 django+mysql，当一次性往 mysql 中写入了近 3w 条数据时，就出现了这个错误

#### 解决方案
调查发现是因为 mysql 默认的有一个写入数据大小限制。只需要在`/etc/my.cnf`(如果没有这个文件就自己建立一个) 下加上下面的代码即可

```conf
[mysqld]
max_allowed_packet = 16M
```

当然，也可以限制一下自己每次往 mysql 中写入数据的数量来解决这个问题。

### vim 分屏与 ctrl p 插件
之前总是听说 `ctrl p` 插件如何如何好用。不过我开始试用的时候，确实没有体会到太强大的地方，知道后来发现 vim 的分屏功能。

试用 vim 分屏功能，加上 ctrl p 实现了不退出 vim 直接打开多个文件的功能，节省了大量在命令行下各种 cd 目录 的时间。

参考：[Vim 的分屏功能](https://coolshell.cn/articles/1679.html)

### 基础正则表达式
有一次，我需要在一个包含很多行短文字的后面补上的空格，将每一行补成相同的宽度。
刚开始使用各种列编辑，块编辑尝试都没有用。难道必须写一个程序来专门处理这件事情？

后来发现正则表达式基本能满足这一要求：

```shell
%s/$/ n_spaces /g
```    

在 vim 中，$符有丰富的含义，这里指的是行尾前一个字符

## 前端
主要记录一下前端学习中遇到的容易混淆的地方，个人感觉在 html，css 和 javascript 中。当属 css 最令人难以捉摸，由于浏览器默认样式或是限制，css 继承，层叠规则等的存在，综合作用之下，很多时候都觉得 css 效果与预期不一致。因此 css 的内容或许会更多一些。
仅仅是零散的记录，暂时没有整理成单独文章的精力。以后或许会把一些可以扩展的地方扩展成文章式的内容。

### CSS 伪类和伪元素区别
伪类：Pseudo-elements
伪元素：Pseudo-classes

#### 概念辨析
stackoverflows 上的回答
[What is the difference between a pseudo-class and a pseudo-element in CSS?](https://stackoverflow.com/questions/8069973/what-is-the-difference-between-a-pseudo-class-and-a-pseudo-element-in-css)

W3C 上的说明
[pseudo-elements](https://www.w3.org/TR/2011/REC-css3-selectors-20110929/#pseudo-elements)
[pseudo-classes](https://www.w3.org/TR/2011/REC-css3-selectors-20110929/#pseudo-classes)

### CSS font-size 相对大小继承
font-size 使用 em 等相对大小时，会发生计算叠加。

> 摘自《css 设计指南》
> 
> 一般来说，em 以浏览器默认值为基本单位，也就是 1em=16px。如果你想使用 em，但又需要设定具体的像素大小，可以把 body 的 font-size 设定为 62.5%。这样，就等于把基准大小从 16 像素改为 10 像素 (16×62.5%=10)。然后，em 与像素的对应关系就十分明确了，比如 1em 等于 10 像素，1.5em 等于 15 像素，2em 等于 20 像素，等等。
> 
> p 元素的文本为 12 像素 (body 的 16 像素基准大小×.75=12px), strong 是 p 的子元素，它的文本相对大小会逐层复合，应该是 16px 0.75 0.75=9px。

**注意与 text-indent 的区别：** font-size 会叠加复合计算，text-indent 则是后面的值覆盖前面的值

> text-indent 可以被子元素继承。如果你 在一个 div 上设定了 text-indent 属性，那么 div 中的所有段落都会继承该缩进值。然而，与 所有继承的 CSS 值一样，这个缩进值并不是祖先元素中设定的值，而是计算的值。下面举一个 例子说明。
> 
> 假设有一个 400 像素宽的 div，包含的文本缩进 5%,则缩进的距离是 20 像素 (400 的 5%)。在这个 div 中有一个 200 像素宽的段落。作为子元素，它继承父元素的 text-indent 值，所以 它包含的文本也缩进。但继承的缩进值是多少呢？不是 5%,而是 20 像素。也就是说，子元素 继承的是根据父元素宽度计算得到的缩进值。结果，虽然段落只有父元素一半宽，但其中的文 本也会缩进 20 像素。这样可以确保无论段落多宽，它们的缩进距离都一样。当然，在子元素 上重新设定 text-indent 属性，可以覆盖继承的值。

### inline-block 的表现

> This value causes an element to generate an inline-level block container. The inside of an inline-block is formatted as a block box, and the element itself is formatted as an atomic inline-level box.
> 大致意思就是：inline-block 后的元素创建了一个行级的块容器，该元素内部（内容）被格式化成一个块元素，同时元素本身则被格式化成一个行内元素。
> 
> 直白一点的意思就是：inline-block 的元素既具有 block 元素可以设置宽高的特性，同时又具有 inline 元素默认不换行的特性。当然不仅仅是这些特性，比如 inline-block 元素也可以设置 vertical-align 属性。简而言之：
> inline-block 后的元素就是一个格式化为行内元素的块容器 ( Block container )

疑问：

inline-block 的 margin 边距会撑开父元素的高度，而普通的 block 的边距则会与父元素的 margin 边距重合？


参考：
[inline-block 前世今生](http://ued.taobao.org/blog/2012/08/inline-block/)

[w3c: the ‘display’ property](https://www.w3.org/TR/CSS2/visuren.html#display-prop)

[mozilla: vertical-align](https://developer.mozilla.org/en-US/docs/Web/CSS/vertical-align)

### rem 使用
今天在 chrome 中实验 rem 相对单位，刚开始的实验代码为：

```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>test rem</title>
	<style type="text/css">
		html{
			font-size: 62.5%;  // 这里的表现有点奇怪
		}
		body{
			font-size: 1.2rem;
		}
		div{
			width: 300px;
			height: 300px;
			background-color: red;
			padding: 1rem;
		}
	</style>
</head>
<body>
	<p>测试字体大小</p>
	<div></div>
</body>
</html>
```

但是这个时候，font-size 并不是 10px，之前在网上看到将 root 元素的 font-size 设置为 10px，方便后文计算。但是在这里似乎并没有起到效果。

原因是因为 chrome 默认所允许的最小字体为 12px，如果设定值小于此值，就会被重置成 12px。这个时候，`1rem=12px`，关于 chrome 最小字体限制，

参考：[Chrome 取消-webkit-text-size-adjust 后的那些事](https://banri.me/web/webkit-text-size-adjust.html)

### 块级元素和行内元素
* 块级元素（block level element）
  * 独占一行
  * 可设置 width，height
  * 默认
* 内联元素（inline element）
  * 设置 width height 无效

参考：[CSS 最核心的几个概念](https://geekplux.com/posts/several_core_concepts_of_css.html)

### width 和 height
定义元素「内容区」的宽度，默认值为 auto 无继承性。浏览器会计算出实际的宽度。

height 和 width 实用百分数相对单位时，相对的是父元素宽度计算（父元素的宽度同理）。如果不指定，默认值就是图片本身的宽高，如果图片加载失败，会显示 alt 属性值，或者浏览器自带的加载失败效果。

### 各种居中方法的使用场景
```css
// 水平居中
// 使用 margin 水平居中元素的时候，
// 需要指定要居中元素的 width 值（与父元素的宽度无关）
// 显然这里只适用于块级元素
// 居中被设置该属性的元素自身，不是子元素，也不是父元素 
margin: 0, auto;   
text-align： center
```

### 如果内部盒大小超出了外部盒所能包含的大小时表现会如何

## 使用过的工具
* reactjs react router
* underscore
* project-path（npm 模块）
* fixed-data-table（react 组件）

## 程序中的过度设计
### 什么是程序中的过度设计
所谓过度设计或者过度工程，顾名思义就是在编写代码的时候，过多的假想了一些本来不存在的场景，并且在这些假想的场景中做出对应的设计，比如处处考虑将来可能出现的变动等等。而实际上，这个时候代码离考虑这些还有很远的距离。

### 我遇到的过度设计
在之前开发产品的时候，实际上在有些地方过度强调了将来可能面临的风险或变动，比如产品中的图片采用 qiniu 云存储，刚开始存储的统一都是 url，但是这时候工程师假想了一个场景：「如果将来七牛的域名改变了，或者将来不适用七牛了怎么办？」然后将原来的统一 url 改为 key，但是发现实际的改写操作并不是一件容易的事情，最后只好作罢。

另一件事情则是产品中的过度设计，在我们开发一个产品的时候，产品经理给出的产品原型变动了无数次，最后的一些变动甚至改变了最基本的数据信息，导致之前写的代码完全失效。而这个产品正是一个需要快速迭代的产品，直到现在第一版还没有做出来。其实这个例子不能算是过度设计，姑且写在这里吧。

不清楚经常强调 mvp(minimum viable product，最小化可行产品) 的我们，为何在做自己的产品的时候，却完全不予考虑了。

当然对于编程经验还不够丰富的我来说，考虑这些还为时尚早，暂且先记录下来，以待将来反思。

## 产品体验
### iOS 初体验
最近服役了三年的 nexus 终于老态龙钟，光荣退伍了。纠结于 android 的多种多样，最终直接选了 iphone。一上手就感觉有几个地方特别的别扭，这里面自然有一些不同系统之前切换带来的生疏感，但是有些地方明显是设计有问题。

#### 鸡肋的下拉菜单
对比 Android 强大的下拉菜单，iOS 的下拉菜单仅用于消息通知或是近期使用的软件功能，而将一部分设置放在了从底部向上拉的菜单中，即便是这些设置也是不完整的，比如 wifi 设置，只能设定开关，而不能由此作为入口进一步设置 wifi。如果想设置 wifi 需要去设置页中去找。

再来看 Android 的下拉菜单设置，不管是 wifi 还是手电筒等功能，都无比的实用与方便，而且使用频率很高。

#### 捉急的闹钟设置
这个就更明显了，android 的闹钟设置一步到位。而 iOS 的闹钟设置页面，要先点击左上角的编辑，进入编辑页，再点击时间才能选择。一步能完成的功能愣是给拆成了好几个步骤。用的时候简直无比纠结。

#### 难以捉摸的输入法和全局赋值粘贴
九宫格输入法中的各个按键占比无比蛋疼，主要区域占比不够，而一个大大的确认按钮放在了右侧，占据了 n 多面积。

再者是输入法和全局复制粘贴的表现简直匪夷所思，它们的各种表现简直难以寻找规律。感觉一会儿是这样，一会儿是那样。

#### 没有充分利用触屏优势的短信界面
ios 短信界面的编辑功能，仍然是左上角有一个编辑按钮，点击之后才能进行编辑。而触摸短信只能进入短信详情，长触摸则没有响应动作。

「编辑按钮」这个功能在 iOS 中随处可见，诸如上面提到的闹钟编辑。这和 Android 有很大的区别，在 Android 中，长触摸一般都表示对触摸项的编辑功能。而 iOS 则没有为长触摸指定动作。这样就又多出一步。实在是没有充分利用触屏本身的交互优势。

### 厨房里的酱油瓶
有一种酱油叫海天，有着一个无比蛋疼的封口设计。打开瓶盖，拉开拉环，倒出香醇的酱油后，你会发现，瓶盖合不上了！酱油这种长期使用产品，不清楚这种汽水拉环一样的一次性设计是为了什么。

而对比厨房里的另一瓶调料——白醋，就明显看出设计上的差距。白醋有一个方便拧开的大瓶盖，为了方便控制倒醋时的流速，还在内部加了一个控制流速的狭缝，可谓方便至极。
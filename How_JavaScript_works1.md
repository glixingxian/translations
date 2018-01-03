#JavaScript是如何工作的：引擎，运行时和调用栈概述
现如今JavaScript越来越流行，很多团队在开发过程的各个层面使用JavaScript : 
前端（Front-End），后端（Back-End），混合式应用（Hybrid Apps），嵌入式设备，
等等。

本篇文章是一系列文章的第一篇，该系列旨在深入探究JavaScript以及它是如何工作的。
我认为如果了解了JavaScript的各个组成部分（building blocks）以及各个部分是
如何一起工作的，我们就可以写出更好的代码和应用程序。
根据[GitHut stats](http://githut.info/)，从活跃代码库（Active repositories）数量和
总得推送(Total pushes)数量来看，JS位居所有语言之首。在其他的数据方面，JS排名也很靠前。
![GitHut stats graph](https://cdn-images-1.medium.com/max/1000/1*Zf4reZZJ9DCKsXf5CSXghg.png)
[查看最新GitHub语言排名](https://madnight.github.io/githut/)

软件项目越来越依赖于JS，也就意味着开发人员必须对这门语言及其生态系统有着更深刻的理解才能做出出色的软件。

但实际情况是，大量的程序员每天都用着JS,但是却对其底层一无所知。
##概述
几乎每个人都听说过V8引擎（V8 Engine）这个概念，并且多数人知道JavaScript是单线程的(single-threaded)而且它有一个回调队列（callback queue）。

在这篇文章中，我们会把这些概念都仔细研究一番，并解释清楚JS实际上是如何工作的。知道了这些细节，你就能合理正确地使用JS提供的API，写出更好的，非阻塞式（non-blocking）程序。

如果你还是一个JS新手，那么这篇文章会帮助你理解为什么相比于其他语言，JS是如此得诡异。

如果你是一个经验丰富的JS程序员，我希望他能给你一些关于JS运行时原理的新感悟。

##JS引擎

谷歌的V8引擎是一个大家比较熟悉的JS引擎。V8引擎被用在Chrome浏览器和Node.js中。下图是一个非常简化的图例
![JS Engine](https://cdn-images-1.medium.com/max/1250/1*OnH_DlbNAPvB9KLxUCyMsA.png)

该引擎由两个部分组成：

* 内存堆栈（Memeory Heap） —— 内存分配（Memory Allocation）发生在这里。
* 调用栈（Call Stack）当代码被执行的时候，所有的栈帧（stack frame）都在这里。

##运行时（The runtime）
有一些浏览器提供的API，比如，setTimeOut，几乎每个JS程序员都用过。但是这些API并不是由JS引擎提供的。

那么，这些API来自哪里呢？

事实上，真实情况更加复杂一些。如下图所示，
![JS runtime](https://cdn-images-1.medium.com/max/1000/1*4lHHyfEhVB0LnQ3HlhSs8g.png)

由此可见，我们除了有一个JS引擎之外，还有其他很多东西。比如那些浏览器提供的，叫做Web API的东西，其中包括DOM，AJAX，setTimeOut等等。

另外还有我们熟悉的事件循环（Event Loop），回调队列（Callback Queue）。

##调用栈（The Call Stack）
JS是一个单线程的程序语言，也就是说，它只有一个调用栈。因此，在同一时间，它只能做一件事。

调用栈是一种数据结构（data structure），主要用来记录目前程序执行所处的位置。如果我们进入（step into）了一个函数（function），我们就把这个函数放到栈顶。如果我们从一个函数返回，我们就把这个函数从栈中弹出（pop off）。以上就是栈所能做的所有事情。
让我们看一个例子。请看下面的代码：

```
function multiply(x, y) {
    return x * y;
}
function printSquare(x) {
    var s = multiply(x, x);
    console.log(s);
}
printSquare(5);

```

当引擎刚要开始执行代码时，调用栈是空的。之后，执行步骤如下所示：
![Call Stack](https://cdn-images-1.medium.com/max/1000/1*Yp1KOt_UJ47HChmS9y7KXw.png)

调用栈中的每一项（Entry）被称为栈帧（Stack Frame）。







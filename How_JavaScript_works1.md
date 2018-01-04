#JavaScript是如何工作的：引擎，运行时和调用栈概述
现如今JavaScript越来越流行，很多团队在开发过程的各个层面使用JavaScript : 
前端（Front-End），后端（Back-End），混合式应用（Hybrid Apps），嵌入式设备，
等等。

本篇文章是一系列文章的第一篇，该系列旨在深入探究JavaScript以及它是如何工作的。
我认为如果了解了JavaScript的各个组成部分（building blocks）以及各个部分是
如何一起工作的，我们就可以写出更好的代码和应用程序。
根据[GitHut stats](http://githut.info/)，从活跃代码库（Active repositories）数量和总的推送(Total pushes)数量来看，JavaScript位居所有语言之首。在其他的数据方面，JavaScript排名也很靠前。
![GitHut stats graph](https://cdn-images-1.medium.com/max/1000/1*Zf4reZZJ9DCKsXf5CSXghg.png)
[查看最新GitHub语言排名](https://madnight.github.io/githut/)

如今软件项目越来越依赖于JavaScript，也就意味着开发人员必须对这门语言及其生态系统有着更深刻的理解才能做出出色的软件。

但实际情况是，大量的程序员每天都用着JavaScript，但是却对其底层一无所知。
##概述
几乎每个人都听说过V8引擎（V8 Engine）这个概念，大多数人知道JavaScript是单线程的(single-threaded)而且它有一个回调队列（callback queue）。

在这篇文章中，我们会把这些概念都仔细研究一番，并解释清楚JavaScript实际上是如何工作的。知道了这些细节，你就能合理正确地使用JavaScript提供的API，写出更好的，非阻塞式（non-blocking）程序。

如果你还是一个JavaScript新手，那么这篇文章会帮助你理解为什么相比于其他语言，JavaScript是如此得诡异。

如果你是一个经验丰富的JavaScript程序员，我希望他能给你一些关于JavaScript运行时原理的新感悟。

##JavaScript引擎

谷歌的V8引擎是一个大家比较熟悉的JavaScript引擎。V8引擎被用在Chrome浏览器和Node.js中。下图是一个非常简化的图例
![JavaScript Engine](https://cdn-images-1.medium.com/max/1250/1*OnH_DlbNAPvB9KLxUCyMsA.png)

该引擎由两个部分组成：

* 内存堆栈（Memeory Heap） —— 内存分配（Memory Allocation）发生在这里。
* 调用栈（Call Stack）—— 当代码被执行的时候，所有的栈帧（stack frame）都在这里。

##运行时（The runtime）
浏览器内置的一些API，比如，setTimeOut，几乎每个JavaScript程序员都用过。但是这些API并不是由JavaScript引擎提供的。

那么，这些API来自哪里呢？

事实上，真实情况要更加复杂一些。如下图所示，
![JavaScript runtime](https://cdn-images-1.medium.com/max/1000/1*4lHHyfEhVB0LnQ3HlhSs8g.png)

由此可见，浏览器除了有一个JavaScript引擎之外，还有其他很多东西。比如那些浏览器提供的，叫做Web API的东西，其中包括DOM，AJAX，setTimeOut等等。

另外还有我们熟悉的事件循环（Event Loop），回调队列（Callback Queue）。

##调用栈（The Call Stack）
JavaScript是一个单线程的程序语言，也就是说，它只有一个调用栈。因此，在同一时间，它只能做一件事。

调用栈是一种数据结构（data structure），主要用来记录目前程序执行所处的位置。如果我们进入（step into）了一个函数（function），我们就把这个函数放到栈顶。如果我们从一个函数返回，我们就把这个函数从栈中弹出（pop off）。以上就是栈所能做的所有事情。
让我们看一个例子。请看下面的代码：

```javascript
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

当有异常（Exception）抛出的时候，堆栈跟踪(Stack trace)也是根据上述过程创建出来的。大体上说，堆栈跟踪就是调用栈在异常抛出那一刻的状态。请看下面的代码：

```javascript
function foo() {
    throw new Error('SessionStack will help you resolve crashes :)');
}
function bar() {
    foo();
}
function start() {
    bar();
}
start();
```

如果上述代码在Chrome中被执行（假设代码位于foo.js这个文件中），我们会得到如下的堆栈跟踪：

![chrome stack trace](https://cdn-images-1.medium.com/max/1000/1*T-W_ihvl-9rG4dn18kP3Qw.png)

"**爆栈（Blowing the stack）**" ——当你使得调用栈深度的最大值时候，就会发生爆栈。这很容易发生，尤其是你在没有对代码测试好的情况下使用递归（recursion）。请看如下代码：

```javascript
function foo() {
    foo();
}
foo();
```

当引擎开始执行代码时，首先会调用“foo”函数，然而这个函数是递归的，它会调用自己，而且没有任何终止条件（termination conditions）。所以同一个函数会在执行过程的每一步被反复地放到调用栈上。看起来就像这样：

![stack recursion without termination](https://cdn-images-1.medium.com/max/1000/1*AycFMDy9tlDmNoc5LXd9-g.png)

最终，在某个时刻，调用栈上的函数调用数量超过了栈最大深度，浏览器这时决定要采取行动，它抛出一个错误，如下图所示：

![call stack overflow error](https://cdn-images-1.medium.com/max/1000/1*e0nEd59RPKz9coyY8FX-uw.png)

只在一个线程上跑代码看似非常简单，因为你不需要处理那些在多线程环境（multi-threaded environments ）中会出现的复杂的情形，比如死锁（dead lock）。

但是只在一个线程上跑代码有很大的限制。由于JavaScript只有一个调用栈，那么在程序执行很慢的时候会出现什么情况呢？

##并发和事件循环（Concurrency & Event Loop）

如果调用栈中有些函数调用需要花费大量的时间，这时候会出现什么情况呢？比如你想要在浏览器中用JavaScript做一些复杂的图像处理工作。

你可能会问了，这怎么可能是个问题？实际上，问题就在于，当调用栈中仍有函数需要执行的时候，浏览器是不能做任何其他的事的 -- 浏览器这时候被阻塞（blocked）了。这就意味着浏览器无法渲染（render），无法执行任何其他的代码，它就这么卡住了（stuck）。如果你想有一个顺畅的UI，那这就是一个大问题。

而且，这不是唯一的问题。如果浏览器开始处理调用栈中的大量任务（task），它就有可能在很长一段时间内停止响应（responsive）。大多数浏览器遇到这种情况时都会报错，然后询问用户是否想要终止（terminate）该页面。

![page unresponsive](https://cdn-images-1.medium.com/max/1000/1*WlMXK3rs_scqKTRV41au7g.jpeg)

嗯，这肯定不是最好的用户体验，对吧？^_^

那么我们应该怎样执行繁重的代码（heavy code）但是又不会阻塞浏览器以至于它变得不再响应呢？嗯，解决方案就是异步回调（asynchronous callback）。

“JavaScript是如何工作的”的第二部分：“ [深入V8引擎 + 写出最优代码的5个小贴士](https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e) ” 会有更加详细的解释。

如果在这期间你发现很难重现或者搞明白一些你的JavaScript应用中的问题，瞅一下[SessionStack](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=Post-1-overview-outro)。 Session会记录你的Web应用中所发生的一切，所有的DOM更改(DOM changes)，用户交互(user interactions)，JavaScript异常(JavaScript exceptions)，堆栈追踪(stack traces)，网络请求失败(failed network requests)，以及调试信息（debug messages）。

有了SessionStack, 你可以以视频的形式重现Web应用中的问题，看到用户使用过程的每个细节。

目前有一个免费的套餐可以让你[免费开始使用](https://www.sessionstack.com/?utm_source=medium&utm_medium=blog&utm_content=Post-1-overview-getStarted)。





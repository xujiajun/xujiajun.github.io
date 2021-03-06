---
layout: post
title: How JavaScript Timers Work [译]
date: 2014-05-11 17:27:31
disqus: y
---

在最基本的层面，重要的是理解JavaScript的定时器是如何工作的。很多时候，他们的行为，因为单线程他们所处让我们先通过检查三项功能，这是我们有机会，可以构建和操作计时器的unintuitively。

var ID = setTimeout的（fn, delay）;启动一个计时器延迟之后，将调用指定的函数。该函数返回一个唯一的ID与该定时器可以在以后的时间被取消。
var ID =的setInterval（fn, delay）; 类似的setTimeout但不断（每次有延迟）调用该函数，直到它被取消。
clearInterval（ID） ，clearTimeout（ID）; 接受（通过上述函数返回），一个计时器ID和发生停止计时器回调。
为了了解定时器是如何工作的内部有一个需要探讨的一个重要的概念：计时器延迟得不到保证。由于在浏览器中的JavaScript的所有单个线程上异步事件（如鼠标点击和计时器）执行时，有一直在执行一个开口只运行。这是最好的证明用的图，如在以下内容：

<img src="/images/427px-Timers.png"></img>

有很多在这个数字信息来消化，但完全理解它会给你一个更好的怎么实现异步JavaScript执行工作。此图是一维：垂直，我们有（挂钟）时间，以毫秒为单位。蓝色方块代表的JavaScript的正在执行的部分。例如，对于约18毫秒，鼠标点击块约11毫秒JavaScript的第一个块执行，依此类推。

由于JavaScript的永远只能执行一个代码块的时间（由于其单线程的性质）每个这些代码块被“堵”其它异步事件的进展。这意味着当一个异步事件发生（如鼠标点击，计时器射击，或者一个XMLHttpRequest完成），它被排队稍后执行（这如何排队实际发生一定从浏览器到浏览器有所不同，所以考虑到这是一个简化）。

首先，JavaScript的第一个块内，两个定时器初始化：一个10ms的setTimeout的和一个10毫秒的setInterval。由于地方和定时器启动时，它实际上触发之前，我们实际上完成的代码的第一个块。但是请注意，它不会立即执行（这是因为线程不能这样做的，）。而不是延迟函数排队，以便在下次有时间来执行。

此外，这个第一个JavaScript块中我们看到了一个鼠标点击发生。与此异步事件关联（我们永远不知道当一个用户可以执行的操作，因此它认为是异步的）JavaScript的回调无法立即从而执行，就像最初的计时器，它正在排队等待稍后执行。

JavaScript的初始块完就立即执行浏览器后，问了一个问题：什么是等待被执行？在这种情况下，两个鼠标点击处理程序和计时器回调都在等待。浏览器，然后挑选一个（鼠标点击回调），并立即执行。定时器将等待，直到下一个可能的时候，以执行。

请注意，当鼠标点击处理程序执行的第一个时间间隔回调函数执行。由于与计时器的处理程序排队等待稍后执行。但是请注意，当间隔再次发射（当定时器处理程序执行），这个时候，处理器的执行被丢弃。如果你是要排队的所有区间回调时的一大块代码的执行结果将是一群与他们之间没有延迟，完成后执行的时间间隔。而不是浏览器往往会简单地等待，直到没有更多的时间间隔处理程序排队更前排队（有问题的区间）。

我们可以，其实，看到这是当第三区间回调火灾时的时间间隔，本身正在执行的情况。这告诉我们一个重要的事实：间隔不关心什么是当前正在执行的，他们会不加区别地排队，即使这意味着回调之间的时间将被牺牲。

最后，在第二区间回调执行完毕后，我们可以看到，什么都不剩了JavaScript引擎执行。这意味着浏览器现在等待一个新的异步事件发生。在50ms标记的地方，当interval再次触发时我们可以看到这样的一个事实。然而这一次，没有什么阻止其执行，所以它会立即触发。

让我们来看看一个例子来更好地说明之间的差异的setTimeout和的setInterval。

{% highlight javascript %}
setTimeout(function(){
    /* Some long block of code... */
    setTimeout(arguments.callee, 10);
  }, 10);
 
  setInterval(function(){
    /* Some long block of code... */
  }, 10);
  {% endhighlight %}

这两段代码可能会出现在功能上等同，乍看之下，但他们不是。特别是setTimeout的代码将始终至少有一个10ms的延迟先前的回调执行后（它可能最终会被更多的，但从来没有少），而的setInterval将尝试执行一个回调不论上次回调被执行的时候每隔10ms。

还有很多，我们在这里学到，让我们回顾一下：

1.JavaScript引擎只有一个线程，迫使异步事件排队等待执行。

2.setTimeout的和的setInterval是在如何执行异步代码根本的不同。

3.如果计时器是从立即执行封锁将被推迟，直到执行（这将是比预期的延迟时间长）的下一个可能的点。

4.如果他们采取足够长的时间来执行（比指定的延时更长）的时间间隔可以执行背到背，没有延迟。

5.所有这一切是建立了令人难以置信的重要的知识。了解如何将JavaScript引擎的工作原理，特别是与大量通常发生异步事件，使得建设先进一段应用程序代码时，一个很好的基础。

by John Resig

from： http://ejohn.org/blog/how-javascript-timers-work/#postcomment


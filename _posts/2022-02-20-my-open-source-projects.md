---
layout: post
title: 那些年我做的开源项目之web篇
disqus: y
---

以下摘自我公众号的内容：


 
今天和你聊聊我之前做过的开源项目那些事。本文是web方向的。

> 如果觉得文章对你有帮助，记得点赞+关注。目前本公众号还没有评论功能，有问题请联系我。

 
![扫码_搜索联合传播样式-白色版](https://user-images.githubusercontent.com/6065007/155329379-c127ea14-b015-4063-b874-7db76d490a10.png)

 

## 前言

本文将按照笔者开发的时间线分享给你，分别是Tastphp、Tastjava、Gorouter，他们分别是用PHP、Java、Go开发，也是笔者在不同时间段对不同语言的使用以及尝试。本文是笔者对过去web方向做的开源项目的一个总结。

## 1 Tastphp

### 背景

这个项目是一个PHP框架。目前我已经没有在维护。当时我做PHP研发已经3年左右，接触过不少框架，包括Codeigniter、Symfony、Laravel、Thinkphp等。当时觉得主流的框架，大而全，很多组件都用不到，并尝试开发自己的框架，想不到仅仅几天就开发了第一版，命名成`Tastphp`，并在公司内部又迭代了几个版本。并于2017年1月份开源（当然开源只是框架，剥离了业务部分的）。
虽然是PHP写的，但是很多思路是共同的，可供你参考。

### 架构设计

Tastphp框架分为[骨架部分 Tastphp skeleton](https://github.com/tastphp/tastphp "Tastphp skeleton")+[内核部分 Tastphp framework core](https://github.com/tastphp/framework "TastPHP framework core")
 
主要讲下内核部分，骨架只是根据需要做了二次封装，加了一些代码分层的目录和console控制台的部分。

核心组件包括DI（依赖注入）、Event（事件）、Router（路由）、Logger（日志）、DBAL（数据抽象层）、Cache（缓存）、Queue（队列）等。

![image](https://user-images.githubusercontent.com/6065007/155329589-572e0e4f-856f-4129-bb6a-727f0cb074f2.png)


**核心思路**

基于`DI（依赖注入）`+ `Event-driven（事件驱动）`。

一个完整的HTTP生命周期，本质上是接受Request请求，对于请求的处理，再返回Response。
所以框架的设计是基于事件驱动的，而依赖注入为了解决依赖其他组件服务的耦合问题。

![image](https://user-images.githubusercontent.com/6065007/155329632-f700af00-d2ed-427e-80f6-c6c134cd5daa.png)


## 2 Tastjava

这个项目是Java开发的Web框架，是2017年8月开源的。

它是基于[Jersey](https://eclipse-ee4j.github.io/jersey/ "Jersey")。其实本质上和Tastphp是同一个东西，只是当时在自学Java，有了这个项目。它的核心组件是`ioc`和`aop`，对应Tastphp就是`DI`和`middleware`。所以我就不展开了，只是时间线上我开发了这个项目，提一下。


## 3 Gorouter

[gorouter](https://github.com/xujiajun/gorouter "Gorouter")是Go开发的一个简单高性能的HTTP router。是在2018年1月开源的。

### 开发动机

这个是我在项目的README中写的[motivation](https://github.com/xujiajun/gorouter#motivation)(动机)
> I wanted a simple and fast HTTP GO router, which supports regexp. I prefer to support regexp is because otherwise it will need the logic to check the URL parameter type, thus increasing the program complexity. So I did some searching on Github and found the wonderful julienschmidt/httprouter: it is very fast，unfortunately it does not support regexp. Later I found out about gorilla/mux: it is powerful as well，but a written benchmark shows me that it is somewhat slow. So I tried to develop a new router which both supports regexp and should be fast. Finally I did it and named xujiajun/gorouter. By the way, this is my first GO open source project. It may be the fastest GO HTTP router which supports regexp, and regarding its performance please refer to my latest Benchmarks.

大致意思：

我想要一个简单而快速并且支持regexp正则表达式的HTTP GO路由器。

我喜欢路由支持regexp，因为否则它将需要逻辑来检查URL参数类型，从而增加了程序的复杂性。

所以我在Github上做了一些搜索，发现了很棒的[julienschmidt/httprouter](https://github.com/julienschmidt/httprouter "julienschmidt/httprouter"):它非常快，不幸的是它不支持regexp。后来我发现了[gorilla/mux](https://github.com/gorilla/mux "mux"):它也很强大，但是一个编写的基准测试显示它有点慢。因此，我试图开发一个既支持regexp又应该速度很快的新路由器。最后我做到了，并命名为[xujiajun/gorouter](https://github.com/xujiajun/gorouter "xujiajun/gorouter")。顺便说一下，这是我的第一个GO开源项目。它可能是最快的GO HTTP路由器支持regexp，关于它的性能，请参考我最新的基准。

### 支持的特性

基本上常用路由功能都实现了，包括参数匹配、路由组、反向路由等。（具体参考[gorouter features](https://github.com/xujiajun/gorouter#features)）

### 设计思路



类似压缩trie的算法。区别于标准的[trie](https://en.wikipedia.org/wiki/Trie "Trie")，会更节省空间。​还有一种是[Radix_tree](https://en.wikipedia.org/wiki/Radix_tree "Radix_tree")，又叫紧凑前缀树，也是一种压缩trie的实现。

下图摘自wiki百科，对trie的演示：

![image](https://user-images.githubusercontent.com/6065007/155329753-7d652405-03e5-4e4f-9c34-72f3134832c5.png)



下面图帮大家演示路由的存储结构：
![image](https://user-images.githubusercontent.com/6065007/155329716-69f3fa4c-5ee9-49fe-b139-7a070aea121e.png)


### 值得提的

1、首先感谢那些贡献者，来贡献这个项目

详见[contributors](https://github.com/xujiajun/gorouter/graphs/contributors)

![image](https://user-images.githubusercontent.com/6065007/155329810-2f98a7f2-255f-43f4-bf4c-edf658cfabdd.png)

 
2、一些Issure：

[特地来说一声感谢](https://github.com/xujiajun/gorouter/issues/55 "特地来说一声感谢")

这位网友专门来感谢我，很开心

![image](https://user-images.githubusercontent.com/6065007/155329839-531c80ec-90ee-4884-a531-eab5ecb92323.png)


[Benchmark is not (only) measuring the routing cost](https://github.com/xujiajun/gorouter/issues/24 "Benchmark is not (only) measuring the routing cost") 

httprouter作者[julienschmidt](https://github.com/julienschmidt)给我提了个issure关于benchmark的问题。
关于这个我想到一个很好的标题，叫《万星大佬给我提了一个issue》，这个标题怎么样？哈哈哈

![image](https://user-images.githubusercontent.com/6065007/155329874-6508aaa2-1a35-41df-a593-4702454c1e14.png)

3、外部发声

"Go夜读"做了分享。
[第 19 期 2018-11-08 如何开发一个简单高性能的 http router 及 gorouter 源码分析](https://talkgo.org/t/topic/38 "第 19 期 2018-11-08 如何开发一个简单高性能的 http router 及 gorouter 源码分析")
 

4、外界评价

[Golang教科书般的web router框架](https://cloud.tencent.com/developer/news/342586 "Golang教科书般的web router框架")

我很怀疑这位同学是我的托，这个标题写的，给我的项目说的这么好。不过话说回来，做为练手项目gorouter还是比较适合学习的。

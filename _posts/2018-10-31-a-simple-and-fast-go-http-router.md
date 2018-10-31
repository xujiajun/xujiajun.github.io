---
layout: post
title: 纪念我的第一个Go开源项目-一个高性能的http路由上GO趋势榜单
date: 2018-10-31 12:27:31
disqus: y
---

## 项目由来
一开始学习 golang 的时候，我是从学习 go 写 web 应用，自然而然需要一个 web 框架或者 web 路由器。

找到这个 julienschmidt/httprouter （有几千 star 吧，截止今天 10 月 31 号，有 7900+star ），

使用了下，发现一般功能有了，但是看了他的 import 库，不支持正则，想改他的代码，发现各种 if，嵌套 walk 有种 bad smell 的感觉，还是放弃。我又试用了另一款著名的路由器 gorilla/mux （也有几千 star 吧，截止今天 10 月 31 号，有 7000+star ），测了下功能比 julienschmidt/httprouter 强大，但是性能差太多。具体见我的 [benchmarks](https://github.com/xujiajun/gorouter#benchmarks)。

于是我决定自己写一个，一来学习下 go，二来也能解决下这个问题。我给自己的目标:

* 0、简单
* 1、测试覆盖率 90%以上，
* 2、支持基本的路由功能，
* 3、支持正则
* 4、性能要高
* 5、文档要完善
* 6、原生 go 实现，不要第三方库

## 项目地址
[https://github.com/xujiajun/gorouter](https://github.com/xujiajun/gorouter)

## 项目原理
用了数据结构压缩 Trie

## Features：
Fast - see benchmarks
* URL parameters
* Regex parameters
* Routes groups
* Custom NotFoundHandler
* Custom PanicHandler
* Middleware Chain Support
* Serve Static Files
* Pattern Rule Familiar
* HTTP Method Get、Post、Delete、Put、Patch Support
* No external dependencies (just Go stdlib)

## 项目情况
* 目前项目已经提交给[awesome-go](https://github.com/avelino/awesome-go)了，已经[被收录](https://github.com/avelino/awesome-go/pull/1820)了，也算给 Go 社区贡献自己小小的力量。希望大家用得上。

* 代码覆盖率 100%。

* examples 里面含有完整例子，方便学习使用，如编写中间件、路由组、路由正则匹配等。

* README 用英文写的，已经完成差不多了，中文如有必要，我再补上。我建议大家看英文
* 当天就上 github GO专栏 趋势榜单， [https://github.com/trending/go?since=daily](https://github.com/trending/go?since=daily)

附件截图：

<img src="https://raw.githubusercontent.com/xujiajun/xujiajun.github.io/master/images/github-go-trend.png">

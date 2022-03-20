---
layout: post
title: 那些年我做的开源项目之KV引擎（NutsDB）
disqus: y
---

> 最新更新：NutsDB目前已经从我个人项目移到nutsdb组织，欢迎开发者共建NutsDB项目！

以下摘自我公众号『太白技术』的内容：

原文：<a href="https://mp.weixin.qq.com/s/jrx9AHt49WP913CxiY7ewQ">https://mp.weixin.qq.com/s/jrx9AHt49WP913CxiY7ewQ</a>

![太白技术公号二维码1](https://user-images.githubusercontent.com/6065007/155888234-397a5416-b9f3-48b0-a471-69bc9778ec82.jpg)


今天和你分享我之前做过的开源项目那些事。
本文的主角是KV引擎：[NutsDB]("https://github.com/xujiajun/nutsdb" "NutsDB")。

Github地址：<a href="https://github.com/xujiajun/nutsdb">https://github.com/xujiajun/nutsdb</a>


## 前言

上一篇[《那些年我做的开源项目之web篇》](https://mp.weixin.qq.com/s/emkUCc1BNiazzuNX1gkcCQ "《那些年我做的开源项目之web篇》")，我和大家分享了web方向开源项目。
本篇将给大家分享我做的开源项目KV引擎NutsDB的故事。

## NutsDB是什么

NutsDB是笔者在2019年1月份开源的，截止今天已经开源三年有余。它是一款简单的、高性能的纯Go语言开发的内嵌型KV引擎，支持基本的Get、Put、Delete操作、TTL，还支持类似Redis的List、Set、Sorted Set，还支持ACID的事务。截止本文发布，最新版本是v0.7.1。

## 为什么有NutsDB

以下是摘自项目的[README-CN.md](https://github.com/xujiajun/nutsdb/blob/master/README-CN.md)。

**注意：**以下是当时开源的时候写的，有些结论可能不一定准确，贴出来只是为了原汁原味。


我想找一个用纯go编写，尽量简单（方便二次开发、研究）、高性能（读写都能快一点）、内嵌型的（减少网络开销）数据库，最好支持事务。因为我觉得对于数据库而言，数据完整性很重要。如果能像Redis一样支持多种数据结构就更好了。 而像Redis一般用作缓存，对于事务支持也很弱。

找到几个备选项：

#### BoltDB
BoltDB是一个基于B+ tree，有着非常好的读性能，还支持很实用的特性：范围扫描和按照前缀进行扫描。有很多项目采用了他。虽然现在官方不维护，由etcd团队在维护 他也支持ACID事务，但是他的写性能不是很好。如果对写性能要求不高也值得尝试。

#### GoLevelDB
GoLevelDB是google开源的leveldb的go语言版本的实现。他的性能很高，特别是写性能，他基于LSM tree实现。可惜他不支持事务(他的README没有提到，其实doc有api的)。

#### Badger
Badger同样是基于LSM tree，不同的是他把key/value分离。据他官网描述是基于为SSD优化。同是他也支持事务。但是我自己测试发现他的写性能没我想象中高，具体见我的benchmark。（使用当时它默认的配置测的）

此外，以上DB都不支持多种数据结构例如list、set等。

### 好奇心的驱使
对于如何实现kv数据库的好奇心吧。数据库可以说是系统的核心，了解数据库的内核或者自己有实现，对更好的用轮子或者下次根据业务定制轮子都很有帮助。

基于以上两点，我决定尝试开发一个简单的kv数据库，性能要好，功能也要强大（至少他们好的功能特性都要继承）。

如上面的选项，我发现大致基于存储引擎的模型分：B+ tree和LSM tree。基于B+ tree的模型相对后者成熟。一般使用覆盖页的方式和WAL（预写日志）来作崩溃恢复。而LSM tree的模型他是先写log文件，然后在写入MemTable内存中，当一定的时候写回SSTable，文件会越来越多，于是他一般作法是在后台进行合并和压缩操作。 一般来说，基于B+ tree的模型写性能不如LSM tree的模型。而在读性能上比LSM tree的模型要来得好。当然LSM tree的模型也可以优化，比如引入BloomFilter。 但是这些模型还是太复杂了。我喜欢简单，简单意味着好实现，好维护，相对不容易出错。

直到我找到bitcask这种模型，他其实本质上也算LSM tree的范畴吧。 他模型非常简单很好理解和实现，很快我就实现了一个版本。但是他的缺点是不支持范围扫描。我尝试去优化他，又开发一个版本，基于B+ tree作为索引，满足了范围扫描的问题。现在这个版本基本上都实现上面提到的数据库的一些有用的特性，包括支持范围扫描和前缀扫描、包括支持bucket、事务等，还支持了更多的数据结构（list、set、sorted set）。

天下没有银弹，NutsDB也有他的局限，比如随着数据量的增大，索引变大，启动会慢，只想说NutsDB还有很多优化和提高的空间，由于本人精力以及能力有限。所以把这个项目开源出来。更重要的是我认为一个项目需要有人去使用，有人提意见才会成长。

## 整体架构

**整体的架构**
 
![image](https://user-images.githubusercontent.com/6065007/159166806-2aeb0e8e-b30c-49c3-a72a-ed3c33e45058.png)

**说明：**

* API：暴露给开发者用的接口，例如：Open()、Put()、Get()、RPush()、SAdd()、ZAdd()等等。
* 内存索引：对各个数据结构的内存索引，包括B+Tree、List、Set、Sorted Set等。
* 读写数据管理：对数据进行持久化的管理。
* 备份、清理：对数据的备份和磁盘冗余的数据清理。
* 事务管理：对数据库读写操作的事务性保障。

## 开源表现

### 开源一天和一周表现

开源**当天**，获得**165个star**；开源**一周**，获得**576个star**

截图来自：[NUTSDB, 高性能内嵌型KV数据库支持事务和多种数据结构](http://xujiajun.cn/2019/03/11/nutsdb/ "NUTSDB, 高性能内嵌型KV数据库支持事务和多种数据结构")



### star增长趋势

后面按照发版一次宣传一下的节奏。截图来自 [https://starchart.cc/xujiajun/nutsdb](https://starchart.cc/xujiajun/nutsdb "xujiajun/nutsdb starchart")

![image](https://user-images.githubusercontent.com/6065007/159166874-2afa2fb3-ab91-4140-acc4-b3846bf36ba6.png)


## 里程碑事件

具体见：<a href="https://github.com/xujiajun/nutsdb/blob/master/CHANGELOG.md">https://github.com/xujiajun/nutsdb/blob/master/CHANGELOG.md</a>

* v0.1.0（2019-2-28）支持Put、Get、Delete、TTL、Range Scanning等
* v0.2.0（2019-3-05）支持List、Set、Sorted Set等
* v0.3.0（2019-3-11）支持sync等
* v0.4.0（2019-3-15）支持mmap方式等
* v0.5.0（2019-11-28）修复一些bug & 支持GetAll()等
* v0.6.0（2021-03-21）支持put带时间戳&支持正则的PrefixSearchScan等
* v0.7.0（2022-03-06) 支持内存模式运行、支持IterateBuckets遍历bucket等
 
## 使用案例

### 被生产环境采用
https://github.com/xujiajun/nutsdb/issues/27

### 被开源项目使用（部分）

内容摘自：[https://github.com/xujiajun/nutsdb/issues/27](https://github.com/xujiajun/nutsdb/issues/27 "Is there any case that is used in production（请问有没有在生产使用的案例）")


* https://github.com/av-elier/nutsdb-cli
   * 第三方开发的NutsDB的cli，作者@av-elier ，俄罗斯人
* https://github.com/cloud-barista/cb-store 
  * cb-store是一个通用的存储库，用于管理Cloud-Barista的Meta信息。您可以选择NUTSDB或ETCD作为cb-store的存储库。作者是韩国的团队
* https://github.com/jrapoport/chestnut
   * chestnut是一个Go实现的加密存储
* https://github.com/ranzhendong/irishman 
  * Irishman 是一个以go为开发语言的中间件，在Kerrigan项目中使用
* https://github.com/bitepeng/b0pass
  * 百灵快传：基于Go语言的高性能 "手机电脑超大文件传输神器"、"局域网共享文件服务器"。斩获了1k+ star
* https://github.com/rule110-io/surge 
  * Surge是一个p2p文件共享应用程序，旨在利用区块链技术实现100%匿名文件传输。Surge是端到端加密、分散和开源的。

#### 更多，大概还有100+项目使用:

详见链接：https://github.com/xujiajun/nutsdb/network/dependents?package_id=UGFja2FnZS0yMjY0ODU0MDM5


## 未来展望

* 易用性：增加cli命令行工具、可视化方向等。
  * 这边特别谢谢新院，他也是NutsDB的用户，提了很多建议给我。所以接下去易用性会放在首位。
* 文档建设：继续完善文档。
* 功能完善：对于已知的功能按照优先级完善，后面对标Redis支持更多功能。
* 分布式：解决高可用、大数据量存储问题。

## 感谢

感谢贡献者、感谢使用者、感谢提issue的网友。如果没有你们的反馈和支持，我可能坚持不下去了。

## 加群

欢迎加群交流，直接扫描下面二维码入群。如果已经过期，加我微信，备注：加群，我拉你进群。
![nutsdb中转_png](https://user-images.githubusercontent.com/6065007/159166824-b2643019-e77d-4124-8981-15502be10106.png)



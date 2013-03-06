---
layout: post
title: "扩展Twitter: 让Twitter快100倍"
description: ""
category: "architecture"
tags: ["all-time-favorites系列", "一日一架构", "架构", "译文"]
---
{% include JB/setup %}

本文翻译自 HighScalability.com的[Scaling Twitter: Making Twitter 10000 Percent Faster](http://highscalability.com/scaling-twitter-making-twitter-10000-percent-faster)一文。译者水平有限，还请多多指正。

**更新 6**： Twiiter的[Evan Weaver](http://blog.evanweaver.com/articles/2009/03/13/qcon-presentation/)提到了一些有意思的改变：所有东西都在内存，数据库只是一个备份；峰值是每秒300tweets；平均每个twee条有126人转发；tweet ID使用向量缓存；行级缓存；段缓存；页缓存；保持独立的缓存；GC让优化Ruby变得很难，所以改用Scala；内部使用Thrift和HTTP; 每一次外部请求意味着100条内部请求；重写了消息队列但保持与原来一样的接口；使用3个队列为对请求进行负载均衡；大量使用A/B测试；为了更快的速度改成使用基于C语言的memcached客户端；优化了关键路径；从网络内存中获取缓存结果而非本地计算。

**更新 5**： Steve Jenson, Alex Payne, Robey Pointer 做了一场名为[Twitter on Scala](http://www.artima.com/scalazine/articles/twitter_on_scala.html)的讨论。这场讨论介绍了为什么Twitter改用Java JVM作为他们的服务框架？为什么他们改用Scala编程？Ruby经常用作前端开发，但是在后端方面它的性能和可靠性还不够。

**更新 4**： Evan Weaver名为[Improving Running Components at Twitter](http://blog.evanweaver.com/articles/2009/03/13/qcon-presentation/)的演讲中讲述了Twitter是如何改进他们的基础设施使得其处理能力由每秒3次请求到139次请求的。他们使用了消息模型，异步处理，3级缓存，以及C与Scala/JVM混合的中间件。

**更新 3**： Gojko Adzic的[Upgrading Twitter without service disruptions](http://gojko.net/2009/03/16/qcon-london-2009-upgrading-twitter-without-service-disruptions/)中包含了很多Twitter新的架构中很好的更新。

**更新 2**： [Twitter Fails Macworld Keynote Test](http://www.techcrunch.com/2008/01/15/twitter-fails-macworld-keynote-test/)的一个批评家指出这篇文章需要更新。我的猜想是这不是一个语言或者架构问题，而是一个不能够添加足够快的硬件到它们的数据中心的问题。这个问题的可预测性是值得商榷的，但是一旦你碰到了，就很难解决它。

**更新**： Twitter发布了[Starling](http://rubyforge.org/projects/starling)-一个轻量级的使用MemCache协议的持久化消息队列。它创建是为了Twitter的后端需要，并且已经在Twitter的线上集群使用。

Twitter开始于一个业余项目，但是它发展很快，从0到百万级浏览量只花了短短的几个月。早期工作得很好的设计决定在面临大量新用户的涌入时开始力不从心。尽管在很早Ruby on Rails就被指出要对扩展问题负责，但是Twitter的首席架构师Blaine Cook认为Ruby一点责任也没有：
> 对于我们来说，这是关于水平扩展的问题。因此，相对于其他语言或者框架来说，Rails and Ruby并不是障碍。换一种"更快"的语言能够帮我们改进10-20%的性能，但是多谢在Ruby and Rails配合下做的架构调整，Twitter比一月份快了100倍。

如果Ruby on Rails没有被质疑，Twitter是怎么学会扩展的如此高的？

##信息源
* Blaine Cook的[Scaling Twitter Video](http://video.google.com/videoplay?docid=-7846959339830379167)
* [Scaling Twitter Slides](http://www.slideshare.net/Blaine/scaling-twitter)
* Rick Denatale写的[Good News](http://talklikeaduck.denhaven2.com/articles/2007/06/22/good-news)博客
* Patrick Joyce的[Scaling Twitter](http://pragmati.st/2007/5/20/scaling-twitter)博客
* [Twitter API Traffic is 10x Twitter’s Site](http://readwritetalk.com/2007/09/05/biz-stone-co-founder-twitter/)
* [A Small Talk on Getting Big. Scaling a Rails App & all that Jazz](http://www.slideshare.net/britt/a-small-talk-on-getting-big-113066)

##平台
* Ruby on Rails
* Erlang
* MySQL
* Mongrel - 小型、快速和安全的混合Ruby/C HTTP服务器
* Munin
* Nagios
* Google Analytics
* AWStats - 实时日志文件分析器
* Memcached

##数字
* 超过350,000个用户。确切的数字通常是非常非常保密的。
* 每秒600次请求
* 每秒平均200-300次连接。高峰期每秒800次连接。
* Mysql每秒处理2,400次请求。
* 180个Rails实例。使用Mongrel作为Web服务器。
* 1个Mysql主服务器(8核)和一个从服务器。从机是只读的。
* 30多个工作处理流程。
* 8台Sun X4100s。
* Rails处理一次请求的时间是200ms。
* 花在数据库的时间平均为50-100ms。
* 超过16G的Memcached。

##架构
* 碰到了非常常见的扩展问题。它出现了很多次。
* 最开始没有监控，没有图，没有统计，这些都让定位和解决问题变得很难。后来使用了Munin和Nagios。它们在Solaris下都很难用。有Google Analytics，但是这些页面没有被加载，所以也没有用。
* 大量使用memcached作为缓存。  
  -- 比如如果获取一个计数很慢的话，你可以只花一微秒将这个计数存储到memcache中。
  -- 获取你朋友的状态是很复杂的。这里面有安全和其他的问题。所以应该是将一个朋友的状态更新到缓存，而不是查询。它从不与数据库打交道。这样的话其响应时间是一个可预测的（最大20ms）。
  -- 活动记录对象之所以不被缓存是由于它们太大了。这些对象将关键属性存储在哈希表中，而当有访问时才延迟加载其他属性。
  -- 90%的请求是API请求。所以不需要在前段做任何页/段缓存。这些页面时效性很好所以缓存没有意义。但是他们缓存API请求。

##未完，待续。

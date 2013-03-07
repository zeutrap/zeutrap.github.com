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
* 消息  
  -- 大量地使用消息。生产者产生消息放置到队列里，然后再分发给消费者。Twitter的主要功能看起来像一个不同格式间的消息网桥（比如SMS, Web和IM等等）。  
  -- 在后台同步发送消息使得朋友的缓存失效，而不是单独地一个个做。  
  -- 一开始我们使用[DRb](http://chadfowler.com/ruby/drb.html)。它基于分布式的Ruby,能够让你通过TCP/IP协议从远端的Ruby对象中发送和接收消息。但是它很脆弱，而且有单点问题。  
  -- 后来使用了[Rinda](http://www.ruby-doc.org/stdlib/libdoc/rinda/rdoc/index.html),他是一个使用元组空间模型的共享队列。但这个队列是个持久的，而且当失败时消息会丢失。  
  -- 尝试过Erlang。但是他的问题是当每时每刻有20,000个用户在等待的时候你怎么样运行一个出问题的服务器？开发者不知道怎么办。也没有文档。所以它违背了你所知道的使用规则。  
  -- 改用了Starling，一个用Ruby写的分布式队列。  
  -- 这个分布式队列通过将数据写入到磁盘来帮我们从系统奔溃中拯救出来。其他大的网页也是将这个简单的方法使用的很好。  
* SMS通过第三方网关提供的API处理。这是非常昂贵的。
* 部署  
  -- 先检查然后推到新的mongrel服务器上去。没有更优雅的方法了。  
  -- 当mongrel服务器被替换时给用户一个内部服务器错误。 
  -- 同时将所有的服务器杀死。之所以不用滚动停止机制是由于消息队列的状态是存储在mongrel里面的，如果我们使用滚动停止，那么剩下的mongrel中的队列将会被塞满。  
* 滥用  
  -- 用户遍历页面并将所有的人都加为好友是导致大部分当机发生的原因。一天内加了9000个好友。这会导致页面崩溃。  
  -- 构建了一个工具来检测这些问题。所以当这些问题发生时能够准确地定位出来。  
  -- 像用户操作一般无情地删除这些行为。  
* 分区  
  -- 未来计划分区。目前还没有。这些变化亦今为止已经足够了。  
  -- 分区的准则是基于时间而不是基于用户。因为大部分的请求是非常时效性的。  
  -- 因为自动内存的缘故，分区很困难。他们不能保证只读操作是真正的只读。有可能你会写一个只读的从机，这是一件很坏的事情。  
* Twitter的API流量十倍于页面流量  
  -- API是Twitter所做的事情里面最重要的。  
  -- 保持服务简单能够让开发者基于Twitter的基础设施想出比Twitter自身更好的点子。比如Twitterrific就是一个拥有不同侧重点的小团队所能创建的使用Twitter的美妙方法。
* Monit被用来杀掉那些太大的进程。

##学到的东西
* 与社区对话。不要隐藏和试图靠自己解决所有问题。在你寻求帮助的时候很多聪明人都愿意来帮助你。
* 像对待商业计划一样对待你的扩展计划。找一些顾问来帮助你。  
* 自己构建系统。 Twitter花了大量的时候来尝试别人的解决方案，虽然这些方案看上去能够工作，但是终究不能。最好是你能够自己构建这些东西，这样的话最起码你能够把控，你也能实现你想要的特性。
* 构建的时候注意用户限制。人们经常会尝试搞垮你的系统。加入合理的限制和检测机制能够保护你的系统不被干掉。
* 别让你的数据库成为末日瓶颈。不是所有的东西都需要大的join。缓存数据。想想其他有创意的方法。[Twitter, Rails, Hammers, and 11,000 Nails per Second](http://www.mooseyard.com/Jens/2007/04/twitter-rails-hammers-and-11000-nails-per-second/)中提到了一个很好的例子。
* 从一开就让你的应用容易切分。这样的话你永远都有办法来扩展你的系统。
* 当你发现你的网站很慢的时候，马上增加报告来跟踪问题。
* 优化数据库  
  -- 将所有东西建上索引。Rails不会为你这样做。 
  -- 使用explain来检查你的查询是如何运行的。索引可能不会像你想的那样工作。  
  -- 一定的反规范化。比如，将用户ID和其朋友ID存在一起，这样避免了大量的代价高的join操作。  
  -- 避免复杂join。  
  -- 避免扫描大的数据集。
* 缓存这该死的一切。个人的活动记录不再缓存。现在这些查询已经足够快了。  
* 测试所有的东西  
  -- 你需要知道当你部署应用的时候它一定能正常工作。  
  -- 有完整的测试套件。所以当缓存出错时能够在事态扩大前找到问题。  
* 长时间运行的进程应当被抽象成守护进程。  
* 使用异常提醒和日常日志来获得及时的问题提醒。这样的话你就能够快速定位问题。  
* 别做傻事  
  -- 改变规模可能会是很愚蠢的。  
  -- 一次加载3000个朋友到内存可能会导致服务器当机。但如果只是4个朋友的话就没什么问题。  
* 大部分的性能问题不是语言引起的，而是应用设计引起的。
* 通过创建API来将你的网站转变成一个开放服务。API是Twitter取得成功的一个很大原因。它让用户能够围绕Twitter创建一个不断扩大、永无止境的微生态系统。你不可能做到你的用户所能做到的一切，你也没有他们那么创新。所以，开放你的应用，让别人能够很容易地整合把你和他们的应用整合在一起。

##相关文章
* 关于分区的问题，你可以看看[Amazon Architecture, An Unorthodox Approach to Database Design : The Coming of the Shard, Flickr Architecture](http://highscalability.com/amazon-architecture)。
* [Mailinator Architecture](http://highscalability.com/mailinator-architecture)中有很多关于防止滥用的好策略。
* [GoogleTalk Architecture](http://highscalability.com/googletalk-architecture)指出了在扩展社交网络站点时好多有趣的论点。



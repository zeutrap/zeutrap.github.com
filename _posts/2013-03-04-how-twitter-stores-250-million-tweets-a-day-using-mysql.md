---
layout: post
title: "Twitter如何利用MySQL每日存储2.5亿条微博"
description: ""
category: "architecture"
tags: ["all-time-favorites系列", "一日一架构", "架构", "译文"]
---
{% include JB/setup %}

本文翻译自 HighScalability.com的[How Twitter Stores 250 Million Tweets A Day Using MySQL](http://highscalability.com/blog/2011/12/19/how-twitter-stores-250-million-tweets-a-day-using-mysql.html)一文。译者水平有限，还请多多指正。

Twitter DBA团队负责人、数据库架构师(Jeremy Cole)[http://www.linkedin.com/in/jeremycole]在O'Reilly MySQL会议上做了一场名为《Twitter大大小小的数据》的演讲。这场演讲的主题是从数据角度来思考Twitter。

他提到了一件有意思的事情：Twitter从原来老的时间[切片](http://highscalability.com/unorthodox-approach-database-design-coming-shard)的方法存储tweet到新的分布式方法的演变。这个方法使用了一个名叫T-Bird的新的Tweet存储，它基于Mysql,建立在[Gizzard](https://github.com/twitter/gizzard)的之上。

##Twwiter原来的消息存储
1. 时间切片当时来说是个好的结构。时间切片的意思是处于同一个日期范围的消息存储在同一个切片上。  
2. 这个问题在于消息会一台接着一台地把机器塞满。  
3. 这是一种很常见的方法，但是有一些缺陷：
  --- 负载均衡。 由于用户通常对新发生的事情比较感兴趣，所以大部分老的机器将不会被访问。这一点在Twitter尤为明显。  
  --- 昂贵。 他会每隔两周就会将一台机器及其对应的从机塞满。这是非常昂贵的。
  --- 逻辑复杂。 每隔两周构建一个全新的集群是DBA团队心中的痛。  

##Twitter全新的存储
1. 消息将会被存储在一个名叫"T-Bird"的内部系统。它是基于Gizzard。二级索引则存储在一个隔离的名叫"T-Flock"的系统，它同样是基于Gizzard。  
2. 每条消息的唯一ID由[Snowflake](https://github.com/twitter/snowflake)生成。这个系统能够均匀地切分到整个集群。FlockDB用来记录ID到ID的映射，存储了不同ID之间的关系(同样使用Gizzard)。  
3. Gizzard是Twitter基于Mysql(InnoDB引擎)的分布式数据存储框架。
  --- 之所以选择InnoDB，是由于它不会损坏数据。Gizzard只是一个数据存储。数据灌入到其中，然后你再取出来。  
  --- 为了在单个节点获得更好的性能，类似于二进制日志和复制等很多特性被关闭了。Gizzard负责处理分发，复制N份数据和任务调度。  
  --- 在Twitter, Gizzard还被其他存储系统用作基础模块。  
4. Gizzard的实现在负载均衡上并不完美，但是它允许：  
  --- **增长缓慢**。不需要去担心机器什么时候会被塞满或者需要每隔固定时间就停止服务。  
  --- **DBA有睡觉的时间**。他们不再需要如此频繁地做决定，也不会随着规模的上升让效率下降。
5. MySQL在大多数时间工作的很好，所以我们用它。相对于特性，Twitter更看重的是稳定，所以我们采用了一个相对老一点的版本。  
6. MySQL不能用来作为ID生成和图存储。  
7. MySQL用来存储小于1.5TB的数据，这是受RAID队列大小所决定的。也可以用作更大数据集的后备存储。  
8. 常用的数据库服务配置是： HP DL380, 72GB RAM, 24 disk RAID10。它在内存和硬盘间做了较好的权衡。  

##MySQL不能用来干所有的事情：
1. **Cassandra** 被用来高速写、低速读。好处是Cassandra能够运行在比Mysql更廉价的机器上。它能够很容易地扩展，推崇schemaless设计。  
2. **Hadoop** 被用来处理亿万行的大规模无结构数据。  
3. **Vertica** 被用来进行分析和大规模的聚合与连接操作，而不必写MapReduce任务。  

##一些其他的想法：
1. **弱耦合**。每当出问题的时候，他们会意识到这是由于做的还不够弱耦合造成的后果。  
2. **软启动**。每次进行发布时注意解耦。特性要能够被关闭或者打开。
3. **开源**。Twiiter对他们的基础架构进行了开源。Jeremy从一种我从未听过的方面来讨论开源。他认为作为一个开发者，开放源代码，意味着你能够认为你的软件永远不会消失。它不是另外一种把软件丢进公司黑洞的做法。它保证你想的与公司开发不一样，这更持久，更有意义。



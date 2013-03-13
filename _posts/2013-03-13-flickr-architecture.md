---
layout: post
title: "Flickr架构"
description: ""
category: "architecture"
tags: ["all-time-favorites系列", "一日一架构", "架构", "译文"]
---
{% include JB/setup %}

本文翻译自 HighScalability.com的[Flickr Architecture](http://highscalability.com/flickr-architecture)一文。

Flickr既是我最喜欢的鸟的名字又是互联网界领先的图片分享站点的名字。Flickr面临着神奇的挑战：他们必须能高性能地处理不断扩充的新内容，不断增加的新用户和源源不断的新功能。他们是怎么做到的？

##信息源
* [Flickr and PHP](http://www.niallkennedy.com/blog/uploads/flickr_php.pdf)(早期文档)  
* [Capacity Planning for LAMP](http://www.kitchensoap.com/talks/MySQLConf2007-Capacity.pdf)  
* Dathan Pattishall的[Federation at Flickr: Doing Billions of Queries a Day](http://mysqldba.blogspot.com/2008/04/mysql-uc-2007-presentation-file.html)  
* Cal Henderson的[Building Scalable Web Sites](http://highscalability.com/book-building-scalable-web-sites)  
* Tim OReilly的[Database War Stories #3: Flickr ](http://radar.oreilly.com/archives/2006/04/database_war_stories_3_flickr.html)  
* [Cal Henderson的演讲](http://www.iamcal.com/talks/)。这里面有大量有用的信息。  

##平台
* PHP  
* MySQL  
* Shards  
* 使用Memcached提供缓存层  
* Squid用来做html和图片的反向代理  
* Linux(RedHat)  
* 使用Smarty生成模板  
* Perl  
* PEAR用来做XML和Email的解析  
* ImageMagick用来做图像处理  
* Java提供节点服务  
* Apache  
* 用SystemImager做部署  
* 用Ganglia做分布式系统监控  
* Subcon将系统配置文件存储到subversion库中简化集群的部署  
* Cvsup用来做网络内部文件集合的分发和更新

##数字
* 每天超过40亿次查询  
* 总计约3500万图片存储在squid缓存中  
* 约200万图片存储在squid内存中  
* 约4.7亿图片  
* memcached的qps为3.8万次请求每秒(总共1200万对象)  
* 2PB原始数据(周日的时候约为1.5TB)  
* 每天增加图片超过40万张  

##架构
* 可以从这个[幻灯片](http://www.slideshare.net/techdude/scalable-web-architectures-common-patterns-and-approaches/138)找到关于Flickr架构的图片。大概描述如下：
  -- 成对的ServerIron  
  ----Squid缓存  
  ------Net App's  
  ----PHP App服务器  
  ------存储管理器  
  ------主从切片  
  ------双树中央数据库  
  ------Memcached集群  
  ------大搜索引擎  
  -- 双树结构是一种MySQL的自定义修改设置，它能够通过增量式增加主服务器来实现扩展而不需要借助环的架构。这样能够廉价地进行扩展，因为相对于需要两倍硬件的主从式来说，它需要的硬件更少。  
  -- 中央数据库包含了像users表之类的数据。users表既包含了主用户键（少量的不同ID），也包含了指向用户数据所在切片的指针。  
* 使用专门的服务器处理静态内容。  
* 支持Unicode。  
* 使用“无共享”架构。  
* 除图片外的所有数据都存储在数据库中。  
* 无状态意味着能够调整服务器上的用户，这样使得API的穿件变得容易。  
* 在第一时间使用复制进行扩展，但这仅仅在读上有帮助。  
* 通过复制可能被搜索的部分数据库来创建搜索结果集。  
* 使用水平扩展，这样的话只需要简单地添加更多的机器就能满足新的需求。  
* 在PHP中解析每封邮件，找出并处理其中的图片。  
* 在早期，他们被主从的延迟所困扰。这里面有太大的负载，而且有但点问题。  
* 他们需要有在线维护、数据修复等等功能，而不需要关闭网站。  
* 关于容量规划有大量很好的材料。可以看看信息源中的材料来获取更多细节。  










---
layout: post
title: "Flickr架构"
description: ""
category: "architecture"
tags: ["all-time-favorites系列", "一日一架构", "架构", "译文"]
---
{% include JB/setup %}

本文翻译自 HighScalability.com的[Flickr Architecture](http://highscalability.com/flickr-architecture)一文。

Flickr既是我最喜欢的鸟的名字又是互联网界领先的图片分享站点的名字。Flickr面临着神奇的挑战：它们必须能高性能地处理不断扩充的新内容，不断增加的新用户和源源不断的新功能。他们是怎么做到的？

##信息源
* [Flickr and PHP](http://www.niallkennedy.com/blog/uploads/flickr_php.pdf)(早期文档)  
* [Capacity Planning for LAMP](http://www.kitchensoap.com/talks/MySQLConf2007-Capacity.pdf)  
* Dathan Pattishall的[Federation at Flickr: Doing Billions of Queries a Day](http://mysqldba.blogspot.com/2008/04/mysql-uc-2007-presentation-file.html)  
* Cal Henderson的[Building Scalable Web Sites](http://highscalability.com/book-building-scalable-web-sites)  
* Tim O'Reilly的[Database War Stories #3: Flickr ](http://radar.oreilly.com/archives/2006/04/database_war_stories_3_flickr.html)  
* [Cal Henderson的演讲](http://www.iamcal.com/talks/)。这里面有大量有用的信息。  

##平台

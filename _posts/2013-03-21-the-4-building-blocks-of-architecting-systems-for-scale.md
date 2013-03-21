---
layout: post
title: "大规模系统架构的四个关键部分"
description: ""
category: "architecture"
tags: ["all-time-favorites系列", "一日一架构", "架构", "译文"]
---
{% include JB/setup %}

本文翻译自 HighScalability.com的[The 4 Building Blocks Of Architecting Systems For Scale](http://highscalability.com/blog/2012/9/19/the-4-building-blocks-of-architecting-systems-for-scale.html)一文。  

如果你在寻找优秀的通用架构原则概述，你可以看看Will Larson的[Introduction to Architecting Systems for Scale](http://lethain.com/introduction-to-architecting-systems-for-scale/)。基于他在Yahoo!和Digg的经验，Will深入地介绍了很多关键理念。下面是一个简短总结：  
1. **负载均衡：可扩展性和冗余**。水平扩展性和冗余通常通过负载均衡来实现，请求的传播会跨越多个资源。  
  -- 智能客户端。客户端拥有一个host列表，它会基于这个列表做负载均衡。这样的好处是对于程序员来说简单。相应的坏处在于很难更新和改变。  
  -- 负载均衡硬件。在大公司，会有专门的负载均衡硬件。它的好处是性能，坏处是价格昂贵和复杂度。  
  -- 负载均衡软件。这是推荐使用的方法。使用软件来处理负载均衡，状态监测等等。  
2. **缓存**。更好地使用你已经拥有的资源，为将要使用的结果提前计算。  
  -- 应用缓存对比数据库缓存。数据库缓存很简单，程序员不需要付出额外劳动。而应用缓存需要显示集成到应用代码里面。  
  -- 内存缓存。性能是最好的。但是通常情况下你拥有的硬盘远远多于内存。  
  -- 内容分发网络。将提供静态内容的负担从你的应用中移除到一个特定的分布式缓存服务中去。  
  -- 缓存失效。缓存是很美好的。但是问题是你必须安全地处理缓存失效的问题。  
3. **离线处理**。处理那些非在线请求。减少延迟，批量处理。  
  -- 消息队列。工作依次排队进入集群的结点以便并行处理。  
  -- 调度定时任务。触发每日，每小时或者其它定时系统任务。  
  -- Map-Reduce。当你的系统对于ad hoc查询变得很大时，你可以开始使用这种特定的数据处理架构。  
4. **平台层**。使用服务层API来去除网页服务器、负载均衡器和数据库中的应用代码。这使得添加新的资源，重用项目基础设施和扩展增长中的组织变得容易。


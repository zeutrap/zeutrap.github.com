---
layout: post
title: "YouTube架构"
description: ""
category: "Architecture"
tags: ["all-time-favorites系列", "架构", "译文"]
---
{% include JB/setup %}

本文翻译自 HighScalability.com的[YouTube Architecture] (http://highscalability.com/youtube-architecture)一文。

YouTube增长很快，现在每日视频的观看次数已经达到1亿次，但其团队人员却只有屈指可数的几人而已。他们是如何将这些视频展示给所有用户的？他们被Google收购后又是如何发展的呢？

##信息源
1. [Google Video](http://video.google.com/videoplay?docid=-6304964351441328559)

##平台
1. Apache
2. Python
3. Linux(Suse)
4. MySQL
5. Psyco, 一个动态的 python->C 编译器
6. Lighttpd，在视频方面Apache的替代者

##数字
1. 支持每天超过1亿次的访问量
2. 创建于2005年2月
3. 2006年3月时每天视频观看量达到3千万
4. 2006年7月时每天视频观看量达到1亿
5. 2个系统管理员，2个软件架构师
6. 2个特性开发工程师，2个网络工程师，1个DBA

##解决快速增长的方法
    while (true)
    { 
        identify_and_fix_bottlenecks();
        drink();
        sleep();
        notice_new_bottleneck();
    }
这个迭代每天进行很多次

##网页服务器
1. 使用NetScalar来进行负载均和静态页面缓存。
2. Apache开启mod_fast_cgi模块。
3. 使用Python应用服务器来处理请求路由。
4. 应用服务器负责与各种数据库和其他信息源通信来获取数据，并格式化html页面；
5. 可以通过增加更多的机器来扩展Web层。
6. Python的代码通常__不是__性能瓶颈，更多的时间是花在了RPC上。
7. Python使得我们可以快速灵活地开发与部署。这对我们面临的竞争很重要；
8. 页面相应时间通常小于100ms。
9. 使用Psyco，这是一种动态的python->c的编译器，它能使用Just-In-Time的编译技术帮助优化内部循环。
10. 对于像加密这种高CPU消耗的需求，我们使用C扩展。
11. 预先生成并缓存渲染代价较高的块。
12. 数据库中使用行级缓存。
13. 完整的Python对象缓存。
14. 一些数据会在计算后发送到每个应用，这样的话它们的值将会被缓存到本地内存中。这是一个**未被充分利用**的策略。最快的缓存是在您的应用服务器里。发送预先计算的数据到每一个服务器并不需要花费太多的时间。你只需要一个代理用来监控数据的变化、预先计算和发送即可。

##视频服务
1. 代价包含带宽、硬件和电力消耗。
2. 每个视频存储在一个迷你集群上。每个视频由多台机器提供服务。
3. 使用集群意味着:  
  ---更多的硬盘意味着更快的速度。  
  ---如果一台机器Down了，其他机器可以替代。  
  ---提供在线备份。  
4. 使用lighttpd作为视频服务器：  
  ---Apache的开销太大。  
  ---使用Epoll来等待多个Fds。  
  ---从单进程切换到多进程可以处理更多的连接数。   
5. 常用的内容迁移到CDN：  
  ---CDN将内容复制到多个地方，这使得内容将可能离用户更近一点，缩短到用户的距离，更容易被用户所访问。  

##*未完，待续*

---
layout: post
title: "Strom-Starter构建失败，缺少twitter4j包 的解决办法"
description: ""
category: 
tags: ["Storm", "Maven", "Twitter4j"]
---
{% include JB/setup %}

原发表于[Zeutrap's Cnblogs](http://www.cnblogs.com/zeutrap/archive/2012/10/11/2720528.html)。

晚上闲得无聊玩了一把storm-starter，在尝试使用maven构建package的时候，总是找不到twitter4j-core 和 twitter4j-stream，报Failure to transfer org.twitter4j:twitter4j-core:2.2.6-SNAPSHOT................

原因是由于Storm-starter使用twitter4j这个仓库来下载twitter4j-core这两个包，而twitter4j已经被伟大的长城盾了。

尝试着使用代理来解决这个问题，由于是在虚拟机环境下，出现了一些问题，未果。

后来在twitter4j的官网上找到了解决办法，修改pom文件从maven主仓库下载即可。

具体做法如下：

修改Storm-Starter的pom文件m2-pom.xml ，修改dependency中twitter4j-core 和 twitter4j-stream两个包的依赖版本，如下：

    <dependency>
        <groupId>org.twitter4j</groupId>
            <artifactId>twitter4j-core</artifactId>
                <version>[2.2,)</version>
    </dependency>
    <dependency>
        <groupId>org.twitter4j</groupId>
            <artifactId>twitter4j-stream</artifactId>
            <version>[2.2,)</version>
    </dependency>
    
原因是原来使用的snapshot版本在中央仓库中没有。

另外，可以将twitter4j.org仓库从配置文件中删除，以加快下载速度。

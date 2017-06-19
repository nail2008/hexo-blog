---
title: Ehcache Monitor 安装及使用
layout: post
date: 2017-05-17 16:47:36
categories:
- 服务端
tags:
- 缓存
- Ehcache
- Ehcache-Monitor
---

[Ehcache](http://www.ehcache.org/)是Java常用的缓存方案，由于直接运行在JVM里，所以还是有很多场合是Redis无法替代的。

监控Ehcache的手段比较单一，只有Ehcache-Monitor，当然也有[自己实现的方法](http://blog.csdn.net/wiker_yong/article/details/52068420)。

查看了不少相关资料，很多链接都不好使了，因为Ehcache的官网改版了。而且似乎刻意去掉了Monitor的信息，官方下载也没有了。

以下是我找到的1.0.3版下载地址：
```
http://terracotta.org/downloads/open-source/destination?name=ehcache-monitor-kit-1.0.3-distribution.tar.gz&bucket=tcdistributions&file=ehcache-monitor-kit-1.0.3-distribution.tar.gz
```
看README.txt这货针对不同Ehcache不同版本使用方法可能有区别，貌似各版本都能用。我使用的是2.5.1，使用正常。

安装方法最详细的还是官方pdf文档，中文的可以看下面两篇：
http://hck.iteye.com/blog/1732660 
http://www.yuananan.cn/html/article/AR64m1684VKax5Ahhp53gp.html

注意注释掉启动文件`ehcache-monitor-kit-1.0.3\bin\startup.bat`里：`-j %PRGDIR%\etc\jetty.xml ^ `行。


官方参考：   
http://www.ehcache.org/documentation/ehcache-2.5.x-documentation.pdf   
http://blog.trifork.com/2009/12/22/using-ehcache-monitor/  







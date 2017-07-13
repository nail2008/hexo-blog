---
title: 基于redis的服务架构（未成文）
layout: post
date: 2017-06-19 16:51:16
categories:
- 服务端
tags:
- redis
- 架构
---
本文探索Redis在企业服务架构中的作用，目前仅收集资料，后期总结心得，逐步完善知识体系。





# 资料

## 架构  

[Redis/Twisted系统整合 ](http://blog.sina.com.cn/s/blog_4680937f0102wsdj.html)  
[时序数据库](https://cloud.baidu.com/product/tsdb.html?track=cp:nsem|pf:pc|pp:tsdb|pu:tsdb|ci:|kw:59248)  

[在云上搭建大规模实时数据流处理系统](http://www.cnblogs.com/langtianya/p/4902461.html)  
[通过Netty通信，采集设备现场GPS数据,并存放在redis服务器](http://www.cnblogs.com/meslog/p/5180510.html)  
[Netty傻瓜教程（五）：不能不谈Redis](https://my.oschina.net/u/438393/blog/845527)  
[Redis在游戏服务器中的应用](http://www.cnblogs.com/agent-k/p/Redis.html)  
[阿里云redis](https://www.aliyun.com/product/kvstore?spm=5176.7114037.746101.1.uqLJwB)  

[redis怎么做消息队列？](https://www.zhihu.com/question/20795043)

## 测试  
[Redis 的性能幻想与残酷现实](http://blog.csdn.net/mindfloating/article/details/50381978)  

知乎问题：  
如何解决redis高并发客户端频繁time out？  
现在业务上每天有5亿+的请求，平时redis的操作在2K+每秒左右。到了高峰有3K+，这时候客户端就会频繁的报connect time out的异常。  
但是，资料上说redis可以达到10W每秒。3K远远不到w这个级别啊，请问有什么建议优化现在的情况么？   

## 使用  
[Jedis对redis的操作详解](http://www.importnew.com/19321.html)  
[Java中使用Jedis操作Redis](http://www.cnblogs.com/liuling/p/2014-4-19-04.html)  

[使用Redis之前5个必须了解的事情](https://www.aliyun.com/zixun/content/2_62_1883753.html)  
[redis3.2新功能--GEO地理位置命令介绍](http://blog.csdn.net/opensure/article/details/51375961)  
[redis 常用整理](http://blog.csdn.net/jone_lu/article/details/50772014)  

## 综合  
[Hello_Nick_Xu的redis系列文章](http://hello-nick-xu.iteye.com/category/314998)  

## 资源  
[各种redis相关项目-oschina](https://www.oschina.net/search?scope=project&q=redis)  


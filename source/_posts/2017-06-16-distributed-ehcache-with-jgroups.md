---
title: Ehcache使用JGroups做组播集群
layout: post
date: 2017-06-16 16:19:05
categories:
- 服务端
tags:
- 缓存
- Ehcache
- JGroups
---
## 关于缓存的几点小想法
开篇先跑跑题。最近在研究集群环境下的缓存同步问题，Web开发领域常见的几种缓存技术：Ehcache、Redis和memcached。  
Ehcache是我们现在用的，它的最大优点是它是纯Java的，跟应用跑在同一个JVM里，不存在传输上的开销；  
Redis在用作缓存时常同memcached对比，我们知道他可以做缓存服务之外，其实他本身是个数据库，甚至自身就可以做消息队列。  

从Ehcache、Redis中的测试报告来看，都说自己更快，这是因为他们都偷换了概念，以己之长比别人的短处。  
Ehcache长处在JVM运行，但这只是单点，跟Redis比单点肯定有优势。免费方案只能考虑做组播，组播对网络环境要求较高，而且如果集群节点很多，组播次数呈级数上升，形成组播风暴，可用性很差。  
Redis单点可能比不过Ehcache，但可以集中或集群式部署缓存服务，供分布式系统中的其他业务服务使用。各个节点采用订阅模式，当某一节点相关缓存更新，通知Redis，Redis在发布给订阅的节点更新缓存。有多少节点就发多少次。

这里不做过多发散，直接说我的小想法——用Ehcache和Redis做二级缓存。  
Ehcache做一级缓存，查询优先从这里查数据，如果没有再去二级缓存Redis上查找，如果还没有，再去数据库查询。

但是很不幸。。已经有人这么做过了——[J2Cache](https://www.oschina.net/p/j2cache)  
思路一致，不过好像太监了。1.3版都没提交中心库。

## Ehcache使用JGroups做组播集群

那既然组播有缺点，为啥还要写这个呢？因为如果你只是做2-3个节点的负载均衡，其实这东西还是靠谱的，又由于很多资料不靠谱，我觉得还是把我梳理的写出来。

跑题结束，下面说正事。

### jar依赖添加

[jgroups官方文档2.1节](http://www.jgroups.org/manual/index.html#Requirements)中有如下内容：  
```
2.1. Requirements
JGroups up to (and including) 3.5.0.Final requires JDK 6.

JGroups 3.6.x to (excluding) 4.0 requires JDK 7.

JGroups 4.0 will require JDK 8.

There is no JNI code present so JGroups should run on all platforms.

Logging: by default, JGroups tries to use log4j2. If the classes are not found on the classpath, it resorts to log4j, and if still not found, it falls back to java.util.logging logger. See Logging for details on log configuration.

```
- 3.6.0（不含）以下的版本需要JDK6   
- 3.6.x 以上到4.0（不含）需要JDK7   
- 4.0以上的需要JDK8  
- 需要log4j或者log4j2

比如，我们的一个项目依赖如下：  
JDK6  
ehcache-core-2.6.11.jar  
jgroups-3.5.1.Final.jar  

另外还需要下面这个jar包，用来整合jgroups和ehcache，点击可以下载：  
[ehcache-jgroupsreplication-1.7.jar](http://mvnrepository.com/artifact/net.sf.ehcache/ehcache-jgroupsreplication/1.7)

最后还有一些jar包：  
slf4j-api-1.6.6.jar  
slf4j-jdk14-1.6.6.jar  
log4j-1.2.13.jar

### 组播策略
组播策略是我们在编写配置文件前需要考虑的问题，我们选择的是`通知失效策略`。  
即各节点独立维护自己的缓存，当一个节点的某缓存发生变化时，并不将改变化同步复制到其他节点。而是通知其他节点清除该缓存。

选择这种策略的原因是我们的应用服务器在Nginx上使用了`IP哈希策略`，一个设备的用户请求会固定由一个应用服务器处理。这种情况下，
很多数据是不用在其他节点缓存具体数据的，只在自己的节点有就行，使用`通知失效策略`减小了同步缓存的开销。

而ehcache可以对不同的数据配置不同的策略，比如`在线用户名单`之类的全局数据我们依然可以使用同步复制的策略，保持各节点下该缓存的数据是一致的。

### 集群策略
一开始我们用了两台机器，不在同一个网段。采用UDP，自动发现的情况下没有问题；但是采用TCP，指定IP的时候就死活不成功，直到挪到同一台，不同端口才成功。（没试同一网段，以后再说。）

### 配置文件
需要修改ehcache配置文件`ehcache.xml`，我还把jgroups的配置文件独立了出来，所以还要新建一个`jgroups_tcp.xml`。  

#### `jgroups_tcp.xml`
官方默认配置例子在jar包里的`tcp.xml`。下面的例子只修改了两个应用的ip及端口（bind_port、initial_hosts配置），如下：  
```xml
<!--
    TCP based stack, with flow control and message bundling. This is usually used when IP
    multicasting cannot be used in a network, e.g. because it is disabled (routers discard multicast).
    Note that TCP.bind_addr and TCPPING.initial_hosts should be set, possibly via system properties, e.g.
    -Djgroups.bind_addr=192.168.5.2 and -Djgroups.tcpping.initial_hosts=192.168.5.2[7800]
    author: Bela Ban
-->
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="urn:org:jgroups"
        xsi:schemaLocation="urn:org:jgroups http://www.jgroups.org/schema/jgroups.xsd">
    <TCP bind_port="40000"
         recv_buf_size="${tcp.recv_buf_size:5M}"
         send_buf_size="${tcp.send_buf_size:5M}"
         max_bundle_size="64K"
         max_bundle_timeout="30"
         use_send_queues="true"
         sock_conn_timeout="300"

         timer_type="new3"
         timer.min_threads="4"
         timer.max_threads="10"
         timer.keep_alive_time="3000"
         timer.queue_max_size="500"

         thread_pool.enabled="true"
         thread_pool.min_threads="2"
         thread_pool.max_threads="8"
         thread_pool.keep_alive_time="5000"
         thread_pool.queue_enabled="true"
         thread_pool.queue_max_size="10000"
         thread_pool.rejection_policy="discard"

         oob_thread_pool.enabled="true"
         oob_thread_pool.min_threads="1"
         oob_thread_pool.max_threads="8"
         oob_thread_pool.keep_alive_time="5000"
         oob_thread_pool.queue_enabled="false"
         oob_thread_pool.queue_max_size="100"
         oob_thread_pool.rejection_policy="discard"/>

    <TCPPING async_discovery="true"
             initial_hosts="${jgroups.tcpping.initial_hosts:localhost[40000],localhost[40001]}"
             port_range="1"/>
    <MERGE3  min_interval="10000"
             max_interval="30000"/>
    <FD_SOCK/>
    <FD timeout="3000" max_tries="3" />
    <VERIFY_SUSPECT timeout="1500"  />
    <BARRIER />
    <pbcast.NAKACK2 use_mcast_xmit="false"
                    discard_delivered_msgs="true"/>
    <UNICAST3 />
    <pbcast.STABLE stability_delay="1000" desired_avg_gossip="50000"
                   max_bytes="4M"/>
    <pbcast.GMS print_local_addr="true" join_timeout="2000"
                view_bundling="true"/>
    <MFC max_credits="2M"
         min_threshold="0.4"/>
    <FRAG2 frag_size="60K"  />
    <!--RSVP resend_interval="2000" timeout="10000"/-->
    <pbcast.STATE_TRANSFER/>
</config>
```

#### `ehcache.xml`

```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:noNamespaceSchemaLocation="ehcache.xsd" updateCheck="false"
		 monitoring="autodetect" dynamicConfig="false">
	<!--start count -->
	<defaultCache maxElementsInMemory="100000" eternal="true"
				  overflowToDisk="false" diskSpoolBufferSizeMB="30" maxElementsOnDisk="10000000"
				  diskPersistent="false" statistics="true"
				  diskExpiryThreadIntervalSeconds="120" memoryStoreEvictionPolicy="LRU">
		<terracotta clustered="false" />
		<!-- 默认采用失效通知策略，各节点独立维护自己的缓存，缓存发生变化时，通知其他节点清除 -->
		<cacheEventListenerFactory
				class="net.sf.ehcache.distribution.jgroups.JGroupsCacheReplicatorFactory"
				properties="replicateAsynchronously=true, replicatePuts=false, replicateUpdates=false,
			replicateUpdatesViaCopy=false, replicateRemovals=true "/>
		<!-- cacheEventListenerFactory属性properties说明 -->
		<!--
		replicateAsynchronously : 对象同步是否异步完成，默认为true。如果比较紧急就设为false。
		                          在一致性时间性要求不强的时候，设为异步可大大提供性能，因为它是异步立即返回的，而且可以批量提交。
		replicateUpdatesViaCopy : 是否将对象变更复制到所有节点，还是只是发送一个失效信息，让对方该缓存失效，当对方需要该缓存时重新计算载入。
		                          默认为true。鉴于对象复制的消耗挺大的，又有锁的问题，而且对方也未必需要该对象，所以此属性建议设为false。
		                          如果业务上真的需要设为true时，就可考虑使用Terracotta了。
		replicatePuts : 增加对象时是否同步，默认为true，如果replicateUpdatesViaCopy为false，选择了失效算法，所以replicatePuts 要设为false。
		replicateUpdates : 修改对象时是否同步，默认为true。
		replicateRemovals : 删除对象时是否同步，默认为true。
		 -->
	</defaultCache>

	<!-- 单独设置菜单的缓存，单个菜单文件100KB，避免上万登录用户全部缓存，一般设定为并发用户数1000-2000 -->
	<cache name="SY_ORG_USER__MENU" maxElementsInMemory="1000"
		   eternal="true" overflowToDisk="false" statistics="true"
		   memoryStoreEvictionPolicy="LRU" >
		<!-- 缓存集群同步策略：各节点独立，不同步 -->
		<cacheEventListenerFactory
		class="net.sf.ehcache.distribution.jgroups.JGroupsCacheReplicatorFactory"
		properties="replicateAsynchronously=true, replicatePuts=false, replicateUpdates=false,
		replicateUpdatesViaCopy=false, replicateRemovals=false "/>
	</cache>

	<!-- 单独设置页面缓存，缓存时间5分钟一刷新 -->
	<cache name="SimplePageCachingFilter" maxElementsInMemory="2000"
		   eternal="false" overflowToDisk="false" timeToIdleSeconds="300"
		   timeToLiveSeconds="300" memoryStoreEvictionPolicy="LFU" >
		<!-- 缓存集群同步策略：各节点独立，不同步 -->
		<cacheEventListenerFactory
				class="net.sf.ehcache.distribution.jgroups.JGroupsCacheReplicatorFactory"
				properties="replicateAsynchronously=true, replicatePuts=false, replicateUpdates=false,
			replicateUpdatesViaCopy=false, replicateRemovals=false "/>
	</cache>

	<!--在线用户-->
	<cache name="ONLINE_USER" maxElementsInMemory="50000"
		   eternal="true" overflowToDisk="false" statistics="true" memoryStoreEvictionPolicy="LFU">
		<!-- 缓存集群同步策略：各节点随时保持同步 -->
		<cacheEventListenerFactory
			class="net.sf.ehcache.distribution.jgroups.JGroupsCacheReplicatorFactory"
			properties="replicateAsynchronously=true, replicatePuts=true, replicateUpdates=true,
			replicateUpdatesViaCopy=true, replicateRemovals=true "/>
	</cache>

	<!-- 集群 JGroup设置 -->
	<cacheManagerPeerProviderFactory class="net.sf.ehcache.distribution.jgroups.JGroupsCacheManagerPeerProviderFactory"
        properties="jgroups_tcp.xml" />

	<!-- ehcache monitor -->
	<!--<cacheManagerPeerListenerFactory class="org.terracotta.ehcachedx.monitor.probe.ProbePeerListenerFactory"-->
	<!--								 properties="monitorAddress=localhost, monitorPort=9889, memoryMeasurement=true"/>-->

</ehcache>
```
### 配置VM OPTIONS
```
-Dfile.encoding=UTF-8           // 解决console中文乱码
-Djgroups.bind_addr=192.168.6.28            // jgroups基本配置
-Djgroups.tcpping.initial_hosts=192.168.6.28[40000]     // jgroups基本配置
-Djava.net.preferIPv4Stack=true  
```

前三个没什么好说的，第四个如果没有，服务启动可能会报错，如下：

```
[2017-05-27 09:55:01.128]<TransferQueueBundler,EH_CACHE,duanyidingdeMacBook-Pro-41155>[WARN ] JGRP000034: duanyidingdeMacBook-Pro-41155: failure sending message to /ff0e:0:0:0:0:8:8:8: java.io.IOException: No route to host (received 7 identical messages from /ff0e:0:0:0:0:8:8:8 in the last 76445 ms) [] org.jgroups.util.SuppressLog.log(SuppressLog.java:47)
```

原因：  
https://gist.github.com/rafaeltuelho/208568668e4205bd9b93  
http://colky.iteye.com/blog/1188408  

### 调试和监控

#### `ehcache-debugger-1.7.1.jar`
http://www.ehcache.org/documentation/2.8/operations/remotedebugger.html  
上面这篇是官方文档，下载到这个jar包，然后执行下面的命令，就可以监控具体缓存了。
```shell
例子：java -jar ehcache-debugger-1.7.1.jar ./../ehcache.xml _CACHE_C_OA_QJ_TYPE_DICT
格式：java -jar ehcache-debugger-1.7.1.jar ehcache配置文件路径 缓存NAME（可选）
```
然后你就可以看到哗哗的日志了。

#### Debug断点
在类JGroupsCacheReceiver的receive方法打断点，可以监控节点获取组播消息的情况。

#### Ehcache-Monitor
安装见：[Ehcache-Monitor](https://nail2008.github.io/2017/05/17/how-to-use-ehcache-monitor/)


## 资料
[JGroups官方文档](http://www.jgroups.org/manual/index.html)

几篇不错的blog：  
http://blog.csdn.net/kindy1022/article/details/6681299  
https://my.oschina.net/u/866380/blog/501082  
http://www.cnblogs.com/fangfan/p/4042823.html  
http://blog.csdn.net/tengdazhang770960436/article/details/49947383  

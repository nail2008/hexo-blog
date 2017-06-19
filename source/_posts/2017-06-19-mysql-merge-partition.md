---
title: MySql merge分表
layout: post
date: 2017-06-19 16:46:36
categories:
- 服务端
tags:
- mysql
- merge
- 数据库分表
---
## MySql的存储引擎

### 常用的几种引擎类型：   
1. InnoDB 是业务系统里最常用的，支持事务处理，不支持全文本搜索
2. MyISAM 性能极高的一个引擎，支持全文本搜索，但不支持事务处理
3. MEMORY 功能等同于MyISAM，但数据存储在内存中，速度更快，适合于临时表

### InnoDB和MyISAM的区别：    
1. MySQL默认采用的是MyISAM。  
2. MyISAM不支持事务，而InnoDB支持。InnoDB的AUTOCOMMIT默认是打开的，即每条SQL语句会默认被封装成一个事务，自动提交，这样会影响速度，所以最好是把多条SQL语句显示放在begin和commit之间，组成一个事务去提交。  
3. InnoDB支持数据行锁定，MyISAM不支持行锁定，只支持锁定整个表。即MyISAM同一个表上的读锁和写锁是互斥的，MyISAM并发读写时如果等待队列中既有读请求又有写请求，默认写请求的优先级高，即使读请求先到，所以MyISAM不适合于有大量查询和修改并存的情况，那样查询进程会长时间阻塞。因为MyISAM是锁表，所以某项读操作比较耗时会使其他写进程饿死。  
4. InnoDB支持外键，MyISAM不支持。    
5. InnoDB的主键范围更大，最大是MyISAM的2倍。   
6. InnoDB不支持全文索引，而MyISAM支持。全文索引是指对char、varchar和text中的每个词（停用词除外）建立倒排序索引。MyISAM的全文索引其实没啥用，因为它不支持中文分词，必须由使用者分词后加入空格再写到数据表里，而且少于4个汉字的词会和停用词一样被忽略掉。   
7. MyISAM支持GIS数据，InnoDB不支持。即MyISAM支持以下空间数据对象：Point,Line,Polygon,Surface等。   
8. 没有where的count(*)使用MyISAM要比InnoDB快得多。因为MyISAM内置了一个计数器，count(*)时它直接从计数器中读，而InnoDB必须扫描全表。所以在InnoDB上执行count(*)时一般要伴随where，且where中要包含主键以外的索引列。为什么这里特别强调“主键以外”？因为InnoDB中primary index是和raw data存放在一起的，而secondary index则是单独存放，然后有个指针指向primary key。所以只是count(*)的话使用secondary index扫描更快，而primary key则主要在扫描索引同时要返回raw data时的作用较大。  

## merge分表策略  

1. 使用merge的话，数据插入策略只能存在FIRST或LAST一张表里，分表策略自然只能选择按时间。
2. mysql单表不要超过500万数据。我们目前巡线轨迹数据30秒一条，一天按12小时算，单用户一天1440条数据。如果按天，单表支持3472个用户；如果按月（31天）单表支持112个用户。单表极限用户数（30秒1条，1天10小时）。当然都是比较理想的情况，实际可以等当前表差不多到500万了，再加新表。

#### 优点  
- 不需要改代码。
- 数据库改动小，也很灵活。
#### 缺点  
- merge只能用在mysql，oracle用的是按范围分区，也可以达到类似的效果。其他数据库暂未研究。

### 一般步骤

1. merge简介分表就是把N条记录的表，分成若干个分表，各个分表记录的总和仍为N。
分表的方法有很多，用merge来分表，是最简单的一种方式.merge是mysql的一种存储引擎，它把一组MyISAM数据表当做一个逻辑单元.
```sql
CREATE TABLE `t` (
  `id`   INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `data` VARCHAR(45)      NOT NULL,
  PRIMARY KEY (`id`)
)
  ENGINE = MERGE
  UNION = (t1, t2)
  INSERT_METHOD = LAST;
```
其中ENGINE = MERGE表示，使用merge引擎。另外ENGINE = MRG_MyISAM是一样的意思。UNION = (t1, t2)表示，挂接了t1,
t2表INSERT_METHOD = LAST表示，插入方式。0不允许插入，FIRST插入到UNION中的第一个表，LAST插入到UNION中的最后一个表.

2. merge数据存储结构mysql中MyISAM引擎下每一张表都对应三个文件: .MYD数据文件，.MYI索引文件，.frm表结构文件.但是Merge引擎下每一张表只有一个.MRG文件.MRG里面存放着分表的关系，以及插入数据的方式。它就像是一个外壳，或者是连接池,
数据存放在分表里面.

### 一个完整的例子
```sql
CREATE TABLE `cdp_gps_record_old` (
  `ID` varchar(40) NOT NULL COMMENT '主键',
  `TASKRECORDID` varchar(40) NOT NULL COMMENT '巡线记录ID，外键',
  `LONGITUDE` decimal(12,9) NOT NULL COMMENT '经度，原始坐标',
  `LATITUDE` decimal(12,9) NOT NULL COMMENT '纬度，原始坐标',
  `X` decimal(20,3) NOT NULL COMMENT '投影X',
  `Y` decimal(20,3) NOT NULL COMMENT '投影Y',
  `SPEED` decimal(8,0) NOT NULL COMMENT '速度',
  `OPM_ID` varchar(40) NOT NULL COMMENT '巡线工ID，外键',
  `TIME` varchar(30) NOT NULL COMMENT '时间',
  `SAVETIME` varchar(30) DEFAULT NULL COMMENT '入库存储时间',
  PRIMARY KEY (`ID`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='实时记录数据old';

CREATE TABLE `cdp_gps_record_now` (
  `ID` varchar(40) NOT NULL COMMENT '主键',
  `TASKRECORDID` varchar(40) NOT NULL COMMENT '巡线记录ID，外键',
  `LONGITUDE` decimal(12,9) NOT NULL COMMENT '经度，原始坐标',
  `LATITUDE` decimal(12,9) NOT NULL COMMENT '纬度，原始坐标',
  `X` decimal(20,3) NOT NULL COMMENT '投影X',
  `Y` decimal(20,3) NOT NULL COMMENT '投影Y',
  `SPEED` decimal(8,0) NOT NULL COMMENT '速度',
  `OPM_ID` varchar(40) NOT NULL COMMENT '巡线工ID，外键',
  `TIME` varchar(30) NOT NULL COMMENT '时间',
  `SAVETIME` varchar(30) DEFAULT NULL COMMENT '入库存储时间',
  PRIMARY KEY (`ID`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='实时记录数据now';

CREATE TABLE `cdp_gps_record` (
  `ID` varchar(40) NOT NULL COMMENT '主键',
  `TASKRECORDID` varchar(40) NOT NULL COMMENT '巡线记录ID，外键',
  `LONGITUDE` decimal(12,9) NOT NULL COMMENT '经度，原始坐标',
  `LATITUDE` decimal(12,9) NOT NULL COMMENT '纬度，原始坐标',
  `X` decimal(20,3) NOT NULL COMMENT '投影X',
  `Y` decimal(20,3) NOT NULL COMMENT '投影Y',
  `SPEED` decimal(8,0) NOT NULL COMMENT '速度',
  `OPM_ID` varchar(40) NOT NULL COMMENT '巡线工ID，外键',
  `TIME` varchar(30) NOT NULL COMMENT '时间',
  `SAVETIME` varchar(30) DEFAULT NULL COMMENT '入库存储时间',
  PRIMARY KEY (`ID`)
) ENGINE = MRG_MyISAM DEFAULT CHARSET=utf8 INSERT_METHOD=LAST COMMENT='实时记录数据总表'
UNION =(`cdp_gps_record_old`,`cdp_gps_record_now`);  

```

## 参考资料
[MyISAM和InnoDB的区别](http://www.cnblogs.com/zhangchaoyang/articles/4214237.html)    
[MyISAM和InnoDB的区别](http://www.cnblogs.com/lyl2016/p/5797519.html)   

[MySQL存储引擎--MyISAM与InnoDB区别](http://blog.csdn.net/xifeijian/article/details/20316775)  

[MySQL使用方案](https://segmentfault.com/a/1190000002675986)    
[如果单表的数据达到300万，是否考虑分表？](https://segmentfault.com/q/1010000002678260)   
[Mysql分表查询引擎Merge技术总结](http://blog.csdn.net/wushangjimo/article/details/40112517)   
[mysql分表方法-----MRG_MyISAM引擎分表法](http://www.cnblogs.com/whoamme/archive/2013/03/13/2958531.html)

[oracle数据分表使用实例](http://www.cnblogs.com/andy6/p/6238512.html)    



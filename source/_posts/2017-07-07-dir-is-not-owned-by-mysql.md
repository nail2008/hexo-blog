---
title: 启动 mysql 失败 Warning:The /usr/local/mysql/data directory is not owned by the 'mysql' or '_mysql'
layout: post
date: 2017-07-07 09:32:48
categories: 
- 数据库
tags:
- mysql
- 启动
- 异常
---
Warning:The /usr/local/mysql/data directory is not owned by the 'mysql' or '_mysql'

这应该是某种情况下导致/usr/local/mysql/data的宿主发生了改变。

解决方法：

打开终端运行 sudo chown -R mysql /usr/local/mysql/data 即可。

mac 下运行  sudo chown -R  _mysql:wheel  /usr/local/mysql/data 。

　　-c 显示更改的部分的信息

　　-f 忽略错误信息

　　-h 修复符号链接

　　-R 处理指定目录以及其子目录下的所有文件

　　-v 显示详细的处理信息


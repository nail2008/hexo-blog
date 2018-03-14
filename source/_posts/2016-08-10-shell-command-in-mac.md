---
title: Shell指令大全(Mac版)
layout: post
date: 2016-08-10 09:56:33
categories: 
- 开发工具
tags: 
- shell
- osx
- mac
---

龟速积累中,按照命令首字母排序

### chmod

##### 说明:
Linux/Unix 的档案调用权限分为三级 : 档案拥有者、群组、其他。利用 chmod 可以藉以控制档案如何被他人所调用。

##### 用法: 
```
 chmod [-cfvR] [--help] [--version] mode file... 
 
 mode : 权限设定字串，格式如下 : [ugoa...][[+-=][rwxX]...][,...]，其中 
 u 表示该档案的拥有者，g 表示与该档案的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。 
 + 表示增加权限、- 表示取消权限、= 表示唯一设定权限。 
 r 表示可读取，w 表示可写入，x 表示可执行，X 表示只有当该档案是个子目录或者该档案已经被设定过为可执行。 
 -c : 若该档案权限确实已经更改，才显示其更改动作 
 -f : 若该档案权限无法被更改也不要显示错误讯息 
 -v : 显示权限变更的详细资料 
 -R : 对目前目录下的所有档案与子目录进行相同的权限变更(即以递回的方式逐个变更) 
 --help : 显示辅助说明 
 --version : 显示版本 
```
##### 例子：
获取该目录机器子目录下所有文件的所有权限：  
```
chmod -R 777 文件或目录     　   
```

递归地给directory目录下所有文件和子目录的属主分配读的权限:  
```
chmod -R u+r directory  
```

##### 延伸阅读:    
http://www.runoob.com/linux/linux-comm-chmod.html 
https://zhidao.baidu.com/question/1667808990925980347.html


### lsof

##### 说明:
lsof（list open file）是一个列出当前系统打开文件的工具。**在linux环境下，任何事物都以文件的形式存在，通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件**。所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等，系统在后台都为该应用程序分配了一个文件描述符，无论这个文件的本质如何，该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口。因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息，因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的。

##### 用法: 
```
lsof  filename 显示打开指定文件的所有进程 
lsof -a 表示两个参数都必须满足时才显示结果 
lsof -c string   显示COMMAND列中包含指定字符的进程所有打开的文件 
lsof -u username  显示所属user进程打开的文件 
lsof -g gid 显示归属gid的进程情况 
lsof +d /DIR/ 显示目录下被进程打开的文件 
lsof +D /DIR/ 同上，但是会搜索目录下的所有目录，时间相对较长 
lsof -d FD 显示指定文件描述符的进程 
lsof -n 不将IP转换为hostname，缺省是不加上-n参数 
lsof -i 用以显示符合条件的进程情况 
lsof -i[46] [protocol][@hostname|hostaddr][:service|port] 
            46 --> IPv4 or IPv6 
            protocol --> TCP or UDP 
            hostname --> Internet host name 
            hostaddr --> IPv4地址 
            service --> /etc/service中的 service name (可以不只一个) 
            port --> 端口号 (可以不只一个) 
```
##### 例子：
- 查看22端口现在运行的情况 

```
lsof -i :22 

```

- 查看root用户进程所打开的txt文件

```
lsof -a -u root -d txt
```
##### 延伸阅读:    
http://www.cnblogs.com/mydomain/archive/2011/09/27/2193560.html
http://www.cnblogs.com/rootq/articles/1401850.html


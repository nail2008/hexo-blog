---
title: 在Mac上安装RabbitMQ
layout: post
date: 2017-07-11 14:24:41
categories: 
- 服务端
tags:
- rabbitmq
- 消息中间件
- mac
---
## 安装
mac下使用Homebrew安装   

```
brew install rabbitmq
```
{% asset_img install.png 安装步骤 %}

默认安装至`/usr/local/opt/rabbitmq`     


## 启动

### 配置自启动
见上图两条命令行：
```
配置开机自启动:
  ln -sfv /usr/local/opt/rabbitmq/*.plist ~/Library/LaunchAgents
然后启动它:
  launchctl load ~/Library/LaunchAgents/homebrew.mxcl.rabbitmq.plist
```

### 手动启动
当然我们的mac不是服务器，开发时使用下面的手动启动就可以了

安装目录下，通过以下命令启动服务
```
sbin/rabbitmq-server
```

启动后服务默认占用5672端口，管理端web在15672端口

## web管理页面

管理页面默认地址：`http://localhost:15672`  
管理端默认账号：guest/guest

注意这个账号是admin权限的。

有了这个图形化界面，创建用户、分权限啥的就不用敲命令了，想了解命令行的使用请查看参考资料，里面有说。




## 参考资料

[官方mac环境安装说明](http://www.rabbitmq.com/install-standalone-mac.html)   
[图解RabbitMQ安装与配置](https://jingyan.baidu.com/article/e4d08ffd9ec61c0fd2f60d1f.html)






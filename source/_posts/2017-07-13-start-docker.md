---
title: 开始使用Docker
layout: post
date: 2017-07-13 14:12:46
categories:
- 服务端
tags:
- docker
- mac
---
## 安装
### Docker for Mac
OSX 10.10以后在Mac上使用Docker变得极简单（不需要装虚拟机及其linux），下载安装即可。

[官网下载](https://store.docker.com/editions/community/docker-ce-desktop-mac?tab=description)

[阿里云下载](http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/?spm=a2c1q.8351553.0.0.468c7ecboTpw2I)

## 使用阿里加速器
1. 注册[阿里云开发者账号](https://account.aliyun.com/)
2. 进入[Docker Hub 镜像站点](https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.12.R6mUIX#/accelerator)
3. 记住`您的专属加速器地址`
4. 打开Docker for Mac。Preferences–>Daemon–>Basic–>Registry mirrors，点+，将你的地址贴到这里。然后Apply&Restart。

## 基本使用
### 查看版本
```
docker version
```
### 搜索可用docker镜像
```
docker search 镜像名字
```
### 下载容器镜像
```
docker pull 用户名/镜像名
```
### 运行容器里的命令
```
docker run 用户名/镜像名 命令........
```
### 在容器中安装新的程序（ubuntu环境）
```
docker run 用户名/镜像名 apt-get install -y 程序名
```
### 查看容器信息
```
docker ps -l
```
查询到的信息如下：  
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
029e4b87152e        learn/tutorial      "apt-get install -..."   37 seconds ago      Exited (0) 33 seconds ago                       dazzling_dijkstra
```

### 保存对容器的修改
```
docker commit 容器ID前3-4位 用户名/新的镜像名
```
这里有一个镜像的基本概念，我们用`docker run`对容器进行操作，有一些会改变它里面的状态、内容。
如果不commit的，下次启动容器，这些状态就丢失了。所以如果有状态变化，后面一定记得commit，保存镜像。
### 检查运行中的镜像
```
docker inspect efe
```
### 发布docker镜像
```
docker push 用户名/镜像名 
```


## 参考资料
[Docker中文](http://www.docker.org.cn/)  
[Docker安装（在Macbook中安装Docker）后配置阿里云加速器](http://blog.csdn.net/hubinqiang/article/details/55113706)
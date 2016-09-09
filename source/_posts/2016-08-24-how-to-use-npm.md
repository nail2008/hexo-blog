---
title: Npm使用
layout: post
date: 2016-08-24 09:05:53
categories: 
- 开发工具
tags: 
- npm
- semver
---
## 网上的教程
网上有很多,不废话了:

- [官方教程](https://docs.npmjs.com/)
- [NPM入了个门](http://www.cnblogs.com/fsjohnhuang/p/4178019.html)
- [菜鸟教程](http://www.runoob.com/nodejs/nodejs-npm.html)

## 常用命令
### npm ls module_name
查看依赖的使用情况  
```
cdp-ui duanyiding$ npm ls jquery
cdp-ui@0.0.1 /Users/duanyiding/WebstormProjects/cdp-ui
`-- rc-queue-anim@0.12.4
  `-- velocity-animate@1.2.3
    `-- jquery@3.1.0 

```

### npm info(view) module_name
列出详细信息，如果只想查看版本：npm info module_name versions  

### npm outdated  
检查包是否已经过时，此命令会列出所有已经过时的包，可以及时进行包的更新   

### npm root -g
查看全局的包的安装路径  

## 网上说得不太清除的几个事儿
### 依赖版本中使用 ~ 和 ^ 的区别  
这个问题其实说的是**语义化版本SemVer**。  
[语义化版本（SemVer）的范围](http://www.u396.com/semver-range.html) 一文是网上搜到的质量较好的中文解释。
但其中对~的用法没说,解释如下:  
假设版本号为:x.y.z  
#### ~  
~x 表示 x.不能变,后边的随便  
~x.y 表示 x.y.不能变,后边的随便  
~x.y.z 表示 x.y.不能变,后边的随便  
~x.y.z-beta1 表示 ~x.y.z-不能变,后边的随便  
#### ^ 
^x.y.z 表示x、y、z中从左往右,第一个非零位往左(包括该非零位)不能变,右边的随便  

关于semver的一些资料:  
http://semver.org/  
https://cnpmjs.org/package/semver   
https://docs.npmjs.com/misc/semver   


---
title: Npm使用
layout: post
date: 2017-09-30 10:34:25
categories: 
- 开发工具
tags: 
- yarn
- npm
---
## 网上的教程

- [中文官方](https://yarn.bootcss.com/)
- [入门](http://www.jianshu.com/p/f05eabdf3ab6)

## 常用命令

### 初始化一个新的项目
```
yarn init
```
### 添加一个依赖包
```
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]
```
### 更新一个依赖包
```
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```
### 删除一个依赖包
```
yarn remove [package]
```
### 安装所有的依赖包
```
yarn
```
or  
```
yarn install
```
### 查看包信息
```
yarn info react
```
### 升级某个全局包
```
yarn global upgrade generator-jhipster
```

## 和 npm 对比
|NPM|YARN|说明|
|:------- |:---------|:------|
|npm init|yarn init	|初始化某个项目|
|npm install/link|	yarn install/link|	默认的安装依赖操作
|npm install taco —save|	yarn add taco	|安装某个依赖，并且默认保存到package.
|npm uninstall taco —save|	yarn remove taco	|移除某个依赖项目
|npm install taco —save-dev|	yarn add taco —dev	|安装某个开发时依赖项目
|npm update taco —save	|yarn upgrade taco	|更新某个依赖项目
|npm install taco --global|	yarn global add taco	|安装某个全局依赖项目
|npm publish/login/logout|	yarn publish/login/logout	|发布/登录/登出，一系列NPM Registry操作
|npm init|	yarn init	|初始化某个项目
|npm run/test	|yarn run/test|	运行某个命令，可以在script脚本中去配置




## 使用淘宝镜像

1. 查看当前的源
```
yarn config get registry
# -> https://registry.yarnpkg.com
```
2. 改成taobao源
```
yarn config set registry 'https://registry.npm.taobao.org'

#yarn config v0.17.3
#success Set "registry" to "https://#registry.npm.taobao.org".
#✨  Done in 0.06s.
```





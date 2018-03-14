---
title: UML类图中的关系
layout: post
date: 2018-03-14 10:16:56
categories:
- 软件设计
tags:
- UML
- Class Diagram
---

在软件设计过程和源码学习时，经常会用到UML类图。
通常对依赖、继承和实现没有什么问题，但正确区分理解关联、组合、聚合的作用和差异常容易搞迷糊。  
需要说明一下，这篇总结并不是入门，是对关键点或是被忽略的内容的一个总结。主要参考wikipedia上的说明，注意英文版和中文版不太一样，某些说法上不统一，以英文版为准。


## UML类图中的关系

{% asset_img umlnotations.png UML Notations %}


### 继承 Inheritance

继承指的是一个类（称为子类、子接口）继承另外的一个类（称为基类、父类、父接口）的功能，并可以增加它自己的新功能的能力。  

需要提一下常看到的另一个词：泛化（Generalization）

如果说继承是子类向父类继承，那么泛化就是与继承的反方向。比如：

```java
Animal a = new Dog();
```
### 实现 Realization/Implementation

没啥好说的，理解下面向接口编程、规约这些词的含义就明白为什么要用接口了。

### 依赖 Dependency

A uses a B 

类A用到了类B，被依赖的对象只是作为一种工具在使用，而并不持有对它的引用。而这种使用关系是具有偶然性、临时性的、单向的、非常弱的，但是B类的变化会影响到A；  
表现在代码层面，为类B作为参数被类A在某个method（方法）中使用。  

```java
// 选课服务
public class EnrollmentService {
    public Record enroll(Student s, Course c){
        Record r = new Record(s,c);
        r.save();
        return r;
    }
}
```

### 关联 Association

A has a B + 只是用到基本没啥从属关系

关联代表一系列的连接。有四种：双向，单向，聚合（包括合成、聚合）和反射。  
双向举个例子：机组和航班。  
其他的后面再说。

```java
public class Order {
    private Customer customer;
}
```

### 聚合 Aggregation

A has a B + 整体——部分 + 但个体独立

聚合是一种关联。聚合是一个较大的“整体”类中包含一个或多个较小的“部分”类的关系。相反，较小的“部分”类是“整个”较大类的一部分。

```java
public class Playlist {
    private List<Song> songs;
}
```

### 组成/组合/合成 Composition

A has a B + 整体——部分 + ownership 同生共死

组成是一种更强大的关联。“整体”负责创建或销毁其“部分”。

```java
public class Apartment {
    private Room bedroom;

    public Apartment() {
        bedroom = new Room();
    }
}
```

## 关联、聚合、组成的区别

这里要强调一点，看了不少资料，只有wikipedia和Stack Overflow上明确表达了上面提到过的关系：聚合、组成是两种程度更紧递紧的关联，如下图。我觉得是靠谱的，在中文资料中基本没提到过这个。

{% asset_img jNyV5.jpg 三者关系图 %}

|   |  聚合  | 组成 |
|:------------- |:---------------| :-------------|
| 生命周期     | 各自拥有各自的生命周期|都使用owner的生命周期 |
| 子对象     | 子对象们都属于一个独立的父对象|子对象们都属于一个独立的父对象|
| 关系 | Has—A    |  Owns |
|性质| collection | mixture |
|举例|车和司机|车和车轮|

最后说下个人的一点体会：  

1. 三者区别主要在强度上，可以用强度高的表述就不用强度低的表述。
2. 聚合和组成如果分不清的时候不要太纠结，不一定一定能从代码层面分清楚，业务上了解关系就可以了。
3. 关联关系，甚至是聚合关系，通常在类图中不会都展示出来，只展示关键的就行了。比如大雁——雁群这种，不画出来关系也很明确。
4. 读代码更重要的是看抽象（继承、实现），区分聚合和组成其实意义不大。



## 资料

[wiki英文](https://en.wikipedia.org/wiki/Class_diagram)   
[wiki中文](https://zh.wikipedia.org/wiki/%E9%A1%9E%E5%88%A5%E5%9C%96)   
[UML实践详细经典教程](http://www.uml.org.cn/oobject/201609092.asp)  
[What is the difference between association, aggregation and composition?](https://stackoverflow.com/questions/885937/what-is-the-difference-between-association-aggregation-and-composition)  
[UML类图java代码实现](http://blog.csdn.net/cjf1002361126/article/details/52750668)  


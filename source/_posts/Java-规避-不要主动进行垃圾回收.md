---
title: Java_规避_不要主动进行垃圾回收
date: 2017-07-28 11:58:48
categories: Java
tags:
    - Java
    - 规避
comments: false
---

ref: [http://blog.csdn.net/p106786860/article/details/9167411](http://blog.csdn.net/p106786860/article/details/9167411)

<!-- more -->

## 建议 
不要调用system.gc，即使经常出现内存溢出也不要调用，内存溢出是可分析的，是可以查找原因的，GC可不是一个好招数。

## 分析 
System.gc主动进行垃圾回收时一个非常危险的动作。因为它要停止所有的响应，才能检查内存中是否有可回收的对象，这对一个应用系统风险极大。

## 场景 
如果一个Web应用，所有的请求都会暂停，等待垃圾回收器执行完毕，若此时堆内存（Heap）中的对象少的话则可以接受，一旦对象较多（现在的Web项目越做越大，框架工具越来越多，加载到内存中的对象就更多了），这个过程非常耗时，可能是0.01秒，也可能是1秒，甚至可能是20秒，这就会严>重影响到业务的正常运行。
又如这样一段代码：new String("abc")，该对象没有任何引用，对JVM来说就是个垃圾对象。JVM的垃圾回收器线程第一次扫描（扫描时间不确定，在系统不繁忙的时候执行）时把它贴上一个标签，说“你是可以给回收的”，第二次扫描时才真正地回收该对象，并释放空间。如果我们直接调用System.gc，就等于说“嗨，你，那个垃圾回收器过来检查一下有没有垃圾对象，回收一下”。程序主动招来了垃圾回收器，这意味着正在运行着的系统要让出资源，以供垃圾回收器执行。


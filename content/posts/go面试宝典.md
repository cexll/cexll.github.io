---
title: "GO面试笔记宝典"
subtitle: "GO面试笔记宝典"
date: 2022-08-14T12:15:53+08:00
lastmod: 2022-08-14T12:15:53+08:00
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [golang]
categories: [golang]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: "MIT"
---

<!--more-->

## 逃逸分析
- 当一个对象的指针被多个方法或者线程引用时,则称这个指针发发生了逃逸. 逃逸分析决定一个变量是分配在堆上还是分配栈上.
- 逃逸分析是怎么完成的, 如果一个函数返回对一个变量的引用, 那么这个变量就会发生逃逸.

## 数据和切片有何异同
- go中切片结构的本质是对数组的封装, 它描述一个数组的片段.

## 切片的容量是怎样增长的
- 当原slice容量小于1024时, 新slice容量为原来的2倍
- 当原slice容量超过1024时, 新slice容量为原来的1.25倍
- go中函数参数传递, 只有值传递, 没有引用传递

## make和new的区别
- make和new都是go语言内置的用来分配内存的函数, 但适用类型不同: 前者适用于slice map channel 等引用类型, 后者使用于 int 数组 结构体等值类型

## map是什么
- 它是由一组<key, value> 对组成的抽象数据结构,并且同一个key只会出现一次.


## map中的key为什么是无序的
- 在map中数量增多 在原有的空间无法保证高效地进行增删改查操作时就会发生扩容, 而map在扩容时会发生key搬移,原来落在同一个bucket中key搬迁后bucket序号加上了2^B, 而遍历的过程在go中并不是固定从0开始, 而是每次都从一个随机序号的bucket开始, 并且从这个bucket的一个随机序号的cell开始遍历
  
## map是线程安全的吗
- 不是, 在查找 赋值 遍历 删除的过程中都会检测写标致, 一旦发现写标志位(等于1), 则会直接panic.
  
## float类型可以作为map的key吗

- 可以, go中只要是可以比较类型都可以作为key. 除了slice map function

## 如何比较两个map是否相等

- 都为nil
- 非空 长度相等, 指向同一个map实体对象
- 相应的key 指向 value "深度"相等

## 可以对map的元素取地址吗
- 不行 (编译报错)

## 可以边遍历边删除吗
- 多线程读写同一个map会panic
- 在同一个协程遍历删除并不会检测同时读写, 理论上是可以的(推荐使用 sync.Map)


## 从一个关闭的通道里仍然能读出数据吗
- 从一个有缓存的channel里依然能读取出有效值, 只有当返回的ok为false时, 读取的数据才无效


## 如何优雅的关闭通道
- 不关闭 等待gc
- 通过中间人来关闭一个额外的信号channel,从而关闭channel

## 通道在什么情况下会引起资源泄露

- channel可能会引发goroutine泄露
  -  goroutine操作channel后, 处于发送或接收阻塞状态, 而channel处于满或空的状态, 一直得不到改变. 同时, 垃圾回收器也不会回收此类资源, 进而导致goroutine会一直处于等待队列中

## 操作channel结构汇总

| 操作 |nil channel | closed channel | not nil, not closed channel |
| --- | --- | --- | --- |
| close | panic | panic | 正常关闭 | 
| 读 <- ch | 阻塞 | 读取到对应类型的零值| 阻塞或正常读取数据. 缓冲型channel为空或非缓冲型channel没有等待的发送者时会阻塞| 
| 写 ch <- | 阻塞 | panic | 阻塞或正常写入数据. 非缓冲型channel没有等待的接受者或缓冲型channel buf 满时会被阻塞| 

## GMP
- G 代表一个goroutine 它包含: 表示goroutine栈的一些字段, 指示当前goroutine的状态, 指示当前运行到的指令地址, 也就是PC值
- M 表示内核线程, 包含正在运行的goroutine等字段
- P 表示一个虚拟的Processor, 它维护一个处理 Runnable 状态的goroutine队列,M需要获取P才能运行G
- 当M没有工作可做的时候, 在它休眠前, 会自旋地来找工作: 检查全局队列, 查看network poller, 试图执行GC任务, 或者去其他P偷G


## 三色标记法
- 白色对象(可能死亡): 未被回收器访问到的对象. 在回收开始阶段, 所有对象均为白色, 当回收结束后, 白色对象均不可达.
- 灰色对象(波面): 已被回收器访问到的对象, 但回收器需要对其中的一个或者多个指针进行扫描, 因为它们可能还指向白色对象.
- 黑色对象(确认存活): 已被回收器访问到的对象, 其中所有字段都已被扫描, 黑色对象中任何一个指针都不可能直接指向白色对象.

> 当垃圾回收开始时, 只有白色对象. 随着标记过程的进行, 灰色对象开始出现(着色), 这时候波面便开始扩大. 当一个对象的所有子节点均完成扫描时, 会被着色为黑色. 当整个堆遍历完成时, 只剩下黑色和白色对象, 这时的黑色对象均为可达对象, 即存活: 而白色对象为不可达对象, 即死亡. 这个过程可以视为以灰色对象为波面, 将黑色对象和白色对象分离, 使波面不断向前推进, 直到所有可达的灰色对象都变为黑色对象为止的过程.



# 引用
本文内容来至Go程序员面试笔记宝典 推荐大家去阅读
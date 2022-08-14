---
title: "Go面经"
subtitle: "Go面经"
date: 2022-07-27T12:15:53+08:00
lastmod: 2022-07-27T12:15:53+08:00
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

# 语言基础
- make和new的差别, 引用类型的意义
- map不初始化使用会怎么样
- map不初始化长度和初始化长度的区别
- map承载多大，大了怎么办
- map的iterator是否安全？能不能一边delete一边遍历？
- 字符串不能改，那转成数组能改吗，怎么改
- 怎么判断一个数组是否已经排序
- 普通map如何不用锁解决协程安全问题
- array和slice的区别
- 零切片、空切片、nil切片是什么
- slice深拷贝和浅拷贝
- map触发扩容的时机，满足什么条件时扩容？
- map扩容策略是什么
- go struct能不能比较？
- map如何顺序读取？
- go中怎么实现set
- Golang 有没有 this 指针？
- Golang 中的引用类型包含哪些?
- 使用range 迭代 map 是有序的吗?
- 解析 JSON 数据时，默认将数值当做哪种类型
- array 类型的值作为函数参数是引用传递还是值传递？
- 在循杯内执行 defer 语句会发生什么?
- switch 中如何强制执行下一个 case 代码块?
- 逃逸分析
- channel的实现
- Goroutine调度策略
- GMP与GC(重点问题: 网络IO等待队列,读写屏障) 
- map的实现 (重点问题: sync.map的实现, map实现随机的方法)
- 高并发下的锁与map读写问题
- 多协程查询切片问题
- 对已经关闭的chan进行读写会怎么样? 为什么?
- 字符串转成byte数组, 会发生内存拷贝吗?
- http包的内存泄露
- channel 是怎么保证线程安全？
- 垃圾回收的过程是怎么样的？
- 什么是写屏障、混合写屏障，如何实现？
- 为什么gc会让程序变慢
- 开多个线程和开多个协程会有什么区别
- 两个interface{} 能不能比较
- 接口是怎么实现的？
- 为什么小对象多了会造成 gc 压力?


# MySQL
- 为什么用b+树不用b树
- 对MVCC的理解
- 幻读是怎么解决的
- redo undo 的作用和实现
- 事务的实现
- 索引怎么建
- 联合索引最左前缀
- 聚簇索引与回表

# Redis
- 基础数据结构
- 底层数据结构实现 (重点问题: 压缩列表)
- 持久化的原理及优化
- 内存淘汰算法实现
- 主从复制原理
- aof与rdb (重点问题: aof重写机制)
- 为什么用跳表
- 分布式锁与RedLock
- 三种分布式的结构
- 大Key

# Mq+ES+分布式
- kafka的零拷贝和顺序IO零拷贝[零拷贝最好说说细节, 用户空间和内核空间mmap]
- kafka的分片, 分片的读一致性和写一致性怎么保证
- ES的倒排索引, 分片的查询召回
- 分布式锁 redis redlock etcd
- 分布式事务 2pc 3pc tcc
- 分布式共识协议 raft和paxos
- 分布式数据库CAP BASE的概念 etcd tidb的了解

# K8s
- k8s的应用和架构
- 监控prometheus
- 微服务架构的内容 (例如服务发现, 链路追踪的工具)

# 算法
- 字符串之实现 Sunday 匹配
- 字符串泄漏之反转字符串(301)
- 字符串中的第一个唯一字符
- 字符串之验证回文串
- 滑动窗口最大值
- 最长公共前缀
- 两个数组的交集
- 最接近的三数之和
# 排序算法
- 冒泡
- 选择


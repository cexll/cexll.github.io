---
title: "通过binlog获取Mysql数据进行恢复同步"
subtitle: ""
date: 2022-01-26T11:02:55+08:00
lastmod: 2022-01-26T11:02:55+08:00
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [mysql, binlog]
categories: [mysql]

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

<!--more-->


> binlog是mysql保存的二进制文件, 可用于恢复数据, 同步数据等一些骚操作,可以有效避免删库跑路等情况

想要`Mysql`自动生成`binlog`需要在配置文件开启
默认编辑`/etc/my.cnf`即可, 具体原因是`mysql`会默认优先读取`/etc/my.cnf`
```bash
mysql --help | grep 'Default options' -A 1
```
执行结果
```bash
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf
```
编辑`/etc/my.cnf`

```conf
[mysqld]
#设置日志路径，注意路经需要mysql用户有权限写
log_bin=/var/lib/mysql/mysql-bin
#设置binlog清理时间
expire_logs_days = 7
#binlog每个日志文件大小
max_binlog_size = 100m
#binlog缓存大小
binlog_cache_size = 4m
#最大binlog缓存大小
max_binlog_cache_size = 512m
server-id=1 
#设置日志三种格式：STATEMENT、ROW、MIXED 。
binlog_format=row
```
修改之后重启`mysql`

# 导出
首先要确定当前使用的是哪一个binlog文件
```mysql
mysql -uroot -p 
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     1259 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
# 找到文件直接可以指定文件名导出
```
## 操作
```
mysqlbinlog /var/lib/mysql/mysql-bin.000001 -v > bak.sql # 这里可以指定导出路径
```
 - --database `指定数据库`
 - -v `row格式的binlog日志`
 - --start-position `指定开始position位置`
 - --stop-position `指定结束position`
 - --start-datetime `指定提取开始时间`
 - --set-charset `转换输出格式`
 - --read-from-remote-server `远程提取`
 - -h `获取更多参数`

# 导入
`SQL`导出之后在另外一个服务器或者本地数据恢复

```
mysql -uroot -p <这里可以指定表> < bak.sql
```

> binlog文件只会对数据有修改的SQL进行保存,所以SELECT是不会保存的

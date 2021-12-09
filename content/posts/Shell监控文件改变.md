---
title: Shell监控文件改变
subtitle: ""
date: 2021-06-24 13:39:48.095
lastmod: 2021-06-24 13:39:48.095
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    shell,监控,linux,inotify
]
categories: [
    linux
]

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






# 使用inotify/fswatch构建自动监控脚本


inotify-tools 是一个用C语言库，一个为Linux提供简单inotify接口的命令行程序。这些程序可以用于监视文件系统事件并执行相应操作。这些程序是用C语言来写的，除了需要Linux内核的inotify支持外，没有其他的依赖。inotify-tools 3.14是目前最新版本，其于2010年3月7日发布。

那么什么inotify又是什么？

inotify，它是Linux在内核 2.6.13 (June 18, 2005)版本中引入的一个新功能，它为用户态监视文件系统的变化提供了强大的支持，允许监控程序打开一个独立文件描述符，并针对事件集监控一个或者多个文件，例如打开、关闭、移动/重命名、删除、创建或者改变属性。

官方站点地址：http://inotify-tools.sourceforge.net/
Github地址：https://github.com/rvoicilas/inotify-tools

## inotify安装

> 只有在内核 2.6.13 (June 18, 2005)以上的Linux版本中才支持inotify-tools。

```bash
Debian/Ubuntu:# sudo apt install inotify-tools
Centos/RHEL:# sudo yum install inotify-tools
```

```bash
# 编译安装
# wget --no-check-certificate https://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
# tar zxvf inotify-tools-3.14.tar.gz
# cd inotify-tools-3.14
# ./configure
# make
# make install
```

> 注：源码包安装需要编译，需要系统已经安装过C编译器

## inotify-tools使用

inotify 的默认内核参数详解

```bash
/proc/sys/fs/inotify/max_queued_events
    默认值: 16384
    该文件中的值为调用inotify_init时分配给inotify instance中可排队的event的数目的最大值，超出这个值得事件被丢弃，但会触发IN_Q_OVERFLOW事件
/proc/sys/fs/inotify/max_user_instances
    默认值: 128
    指定了每一个real user ID可创建的inotify instatnces的数量上限
/proc/sys/fs/inotify/max_user_watches
    默认值: 8192
    指定了每个inotify instance相关联的watches的上限，也就是每一个inotify实例可监控的最大目录数。如果监控的文件数目巨大，需要根据实际情况适当增加此值得大小。

注意:
    max_queued_events 是 Inotify 管理的队列的最大长度，文件系统变化越频繁，这个值就应该越大！如果你在日志中看到Event Queue Overflow，说明max_queued_events太小需要调整参数后再次使用
```

优化参数配置

```bash
echo 104857600 > /proc/sys/fs/inotify/max_user_watches
```

inotify-tools 工具包中包含了两个命令

* inotifywait
* inotifywatch

## inotifywait

>inotifywait 仅执行阻塞，等待 inotify 事件，你可以使用它来监控任何一组文件和目录，或监控整个目录树（目录、子目录、子目录的子目录等等），并且可以结合 shell 脚本，更好的使用 inotifywait。

命令格式：

```bash
inotifywait [-hcmrq] [-e <event> ] [-t <seconds> ] [--format <fmt> ] [--timefmt <fmt> ] <file> [ ... ]
```

选项参数：

```bash
-h|--help     	显示帮助信息
@<file>       	排除不需要监视的文件，可以是相对路径，也可以是绝对路径
--exclude <pattern>
                正则匹配需要排除的文件，大小写敏感
--excludei <pattern>
                正则匹配需要排除的文件，忽略大小写。
-m|--monitor  	接收到一个事情而不退出，无限期地执行。默认行为是接收到一个事情后立即退出
-d|--daemon   	跟--monitor一样，除了是在后台运行，需要指定--outfile把事情输出到一个文件。也意味着使用了--syslog
-r|--recursive	监视一个目录下的所有子目录
--fromfile <file>
                从文件读取需要监视的文件或排除的文件，一个文件一行，排除的文件以@开头
-o|--outfile <file>
                输出事件到文件.
-s|--syslog   	输出错误信息到系统日志
-q|--quiet    	不输出详细信息，只输出事件
-qq           	除了致命错误，不会输出任何信息
--timefmt <fmt>	指定时间格式，用于�format选项中的%T格式
-c|--csv      	输出csv格式。
-t|--timeout <seconds>
                设置超时时间，如果为0，则无限期地执行下去。
-e|--event <event1> [ -e|--event <event2> ... ]
                指定监听的时间，如果省略，则侦听所有事件。
--format <fmt>	指定输出格式
     %w 表示发生事件的目录
     %f 表示发生事件的文件
     %e 表示发生的事件
     %Xe 事件以“X”分隔
     %T 使用由--timefmt定义的时间格式
```

可监听的事件

```bash
access		    文件或者目录被读
modify		    文件或目录被写入
attrib		    文件或者目录属性被更改
close_write	  文件或目录关闭，在写模式下打开后
close_nowrite	文件或目录关闭，在只读模式打开后
close		      文件或目录关闭，而不管是读/写模式
open		      文件或目录被打开
moved_to	    文件或者目录移动到监视目录
moved_from	  文件或者目录移出监视目录
move		      文件或目录移出或者移入目录
create		    文件或目录被创建在监视目录
delete		    文件或者目录被删除在监视目录
delete_self	  文件或目录移除，之后不再监听此文件或目录
unmount		    文件系统取消挂载，之后不再监听此文件系统
```

示例1、监控/data目录

```bash
inotifywait -rmq /data
```

我们在另一个终端中向该目录中写入一个文件

```bash
echo "test" >> /data/newfile
```

这个时候我们就会在前一个终端中看到如下信息

```bash
# inotifywait -rmq /data
/data/ CREATE newfile
/data/ OPEN newfile
/data/ MODIFY newfile
/data/ CLOSE_WRITE,CLOSE newfile
```

如上所示，我们监控的了对于newfile文件的 CREATE、OPEN、MODIFY、CLOSE_WRITE、CLOSE等事件。

示例2、实时监控对/etc/passwd 文件的修改、删除和权限相关时间，并且按照指定格式输出。

```bash
# inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format  '%T %w%f %e' --event modify,delete,attrib  /etc/passwd
```

这是我们在另一终端创建一个新用户

```bash
# useradd testuser
```

这时在前一个终端中就会监控到一个ATTRIB 事件

```bash
# inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format  '%T %w%f %e' --event modify,delete,attrib  /etc/passwd
29/10/16 16:59 /etc/passwd ATTRIB
```

示例3、实现对 /data/web 目录进行监控，监控文件删除，修改，创建和权限相关事件，并且要求将监控信息写入/var/log/web_watch.log。要求日志条目要清晰明了，能突显文件路径、事件名和时间。

```bash
# cat web_watch.sh
#!/bin/bash
inotifywait -mrq --timefmt '%y/%m/%d %H:%M' --format  '%T %w%f %e' --event delete,modify,create,attrib  /data/web | while read  date time file event
  do
      case $event in
          MODIFY|CREATE|MOVE|MODIFY,ISDIR|CREATE,ISDIR|MODIFY,ISDIR)
                  echo $event'-'$file'-'$date'-'$time >> /var/log/web_watch.log
              ;;

          MOVED_FROM|MOVED_FROM,ISDIR|DELETE|DELETE,ISDIR)
                  echo $event'-'$file'-'$date'-'$time /var/log/web_watch.log
              ;;
      esac
  done
```

运行脚本后，在另一个终端操作后，查看/var/log/web_watch.log日志

```bash
# cat /var/log/web_watch.log
CREATE-/data/web/a-14/06/27-16:21
CREATE-/data/web/aa-14/06/27-16:21
CREATE-/data/web/aaaa-14/06/27-16:24
CREATE-/data/web/aaaaa-14/06/27-16:24
```

更多的使用方式，请查看 inotifywatch man page

## inotifywatch

> inotifywatch 用来收集关于被监视的文件系统的统计数据，包括每个 inotify 事件发生多少次。

命令格式：

```bash
inotifywatch  [-hvzrqf]  [-e  <event>  ] [-t <seconds> ] [-a <event> ] [-d <event> ] <file> [ ... ]
```

选项参数：

```bash
-h|--help    	            显示帮助信息
-v|--verbose 	            详细信息
@<file>       	          排除不需要监视的文件，可以是相对路径，也可以是绝对路径
--fromfile <file>         从文件读取需要监视的文件或排除的文件，一个文件一行，排除的文件以@开头
--exclude <pattern>       正则匹配需要排除的文件，大小写敏感
--excludei <pattern>      正则匹配需要排除的文件，忽略大小写。
-z|--zero                 输出表格的行和列，即使元素为空
-r|--recursive	          监视一个目录下的所有子目录
-t|--timeout <seconds>
                          设置超时时间，如果为0，则无限期地执行下去。
-e|--event <event1> [ -e|--event <event2> ... ]
                          指定监听的时间，如果省略，则侦听所有事件。
-a|--ascending <event>    以指定事件升序排列
-d|--descending <event>   以指定事件降序排列
```

示例1、统计/data目录所在文件系统发生的事件次数

```bash
inotifywatch -v -e create,modify,delete -t 30 -r /data
```

然后在另一终端中进行一些操作

```bash
# echo "test" >> /data/newfile1
...
# rm /data/newfile
rm：是否删除普通文件 "/data/newfile"？y
# rm /data/newfile1
rm：是否删除普通文件 "/data/newfile1"？y
```

30秒后，前一个终端会生成如下报告

```bash
# inotifywatch -v -e create,modify,delete -t 30 -r /data
Establishing watches...
Setting up watch(es) on /data
OK, /data is now being watched.
Total of 1 watches.
Finished establishing watches, now collecting statistics.
Will listen for events for 30 seconds.
total  modify  create  delete  filename
11     8       1       2       /data/
```

更多的使用方式，请查看 inotifywatch man page
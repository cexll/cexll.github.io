---
title: Linux系统高并发socket最大连接数优化
subtitle: ""
date: 2021-06-24 13:41:13.106
lastmod: 2021-06-24 13:41:13.106
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    linux,高并发,优化
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





> 参考 https://lanjingling.github.io/2016/05/10/linux-socket-youhua1/

*可以通过ulimit –a命令来查看系统的一些资源限制情况，如下：*

```bash
# ulimit -a
core file size          (blocks, -c) 1024
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 127422
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 20480
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) unlimited
cpu time               (seconds, -t) unlimited
max user processes              (-u) 81920
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

这里重点关注open files和max user processes。分别表示：单个进程打开的最大文件数；系统可以申请最大的进程数。

1. 查看、修改文件数（当前session有效）：

```bash
# ulimit -n
20480
# ulimit -n  20480
```

2. 查看、修改进程数（当前session有效）：

```bash
# ulimit -u
81920
# ulimit -u  81920
```

3. 永久设置文件数、最大进程：

- 可以编辑# vim /etc/security/limits.conf在其中指定最大设置；
- 或者在/etc/profile文件指定；

## 一、最大进程数：

最近在Linux服务器上发布应用时碰到一个如下的异常：

```
Caused by: java.lang.OutOfMemoryError: unable to create new native thread
at java.lang.Thread.start0(Native Method)
at java.lang.Thread.start(Thread.java:640)
```

初看可能会认为是系统的内存不足，如果这样想的话就被这段提示带到沟里面去了。上面这段错误提示的本质是Linux操作系统无法创建更多进程，导致出错。因此要解决这个问题需要修改Linux允许创建更多的进程。

1、临时设置：

我们可以使用 ulimit -u 81920 修改max user processes的值，但是只能在当前终端的这个session里面生效，重新登录后仍然是使用系统默认值。

2、永久设置：

```bash
1. 编辑
# vim /etc/security/limits.conf
```

在文件中添加如下内容：

```bash
soft nproc 81920
hard nproc 81920
注：*表示所有用户，soft、hard表示软限制、硬限制。（软限制<=硬限制）
```

2）或者在/etc/profile文件中添加：

```bash
ulimit -u 81920
```

这样每次用户登录就可以设置最大进程数。

## 二、最大打开文件数：

最大文件打开数在Linux平台上，无论编写客户端程序还是服务端程序，在进行`高并发TCP连接处理时，最高的并发数量都要受到系统对用户单一进程同时可打开文件数量的限制`(这是因为系统为每个TCP连接都要创建一个socket句柄，每个socket句柄同时也是一个文件句柄)。

1、查看最大打开文件数：

```bash
# ulimit -n
1024
```

这表示当前用户的每个进程最多允许同时打开1024个文件，这1024个文件中还得除去每个进程必然打开的标准输入，标准输出，标准错误，服务器监听 socket，进程间通讯的unix域socket等文件，那么剩下的可用于客户端socket连接的文件数就只有大概1024-10=1014个左右。也就是说缺省情况下，基于Linux的通讯程序最多允许同时1014个TCP并发连接。

对于想支持更高数量的TCP并发连接的通讯处理程序，就必须修改Linux对当前用户的进程同时打开的文件数量的软限制(soft limit)和硬限制(hardlimit)。其中：

- 软限制是指Linux在当前系统能够承受的范围内进一步限制用户同时打开的文件数；
- 硬限制则是根据系统硬件资源状况(主要是系统内存)计算出来的系统最多可同时打开的文件数量。`通常软限制小于或等于硬限制`。

2、修改最大打开文件数：

```bash
# ulimit -n 10240
```

上述命令中，临时设置的单一进程允许打开的最大文件数（当前session有效）。 如果系统回显类似于“Operation notpermitted”之类的话，说明上述限制修改失败，实际上是因为在中指定的数值超过了Linux系统对该用户打开文件数的软限制或硬限制。因此，就需要修改Linux系统对用户的关于打开文件数的软限制和硬限制。

1）首先，修改/etc/security/limits.conf文件，在文件中添加如下行：

```bash
speng soft nofile 10240
speng hard nofile 10240
```

其中speng指定了要修改哪个用户的打开文件数限制，`可用’*’号表示修改所有用户的限制`；soft或hard指定要修改软限制还是硬限制；10240则指定了想要修改的新的限制值，即最大打开文件数(请注意软限制值要小于或等于硬限制)。修改完后保存文件。

2）其次，修改/etc/pam.d/login文件，在文件中添加如下行：

```bash
session required /lib/security/pam_limits.so
```

这是告诉Linux在用户完成系统登录后，应该调用pam_limits.so模块来设置系统对该用户可使用的各种资源数量的最大限制(包括用户可打开的最大文件数限制)，`而pam_limits.so模块就会从/etc/security/limits.conf文件中读取配置来设置这些限制值`。修改完后保存此文件。


3）第三步，查看Linux系统级的最大打开文件数限制（硬限制），使用如下命令：

```bash
# cat /proc/sys/fs/file-max
12158
```

这表明这台Linux系统最多允许同时打开(即包含所有用户打开文件数总和)12158个文件，是Linux系统级硬限制，所有用户级的打开文件数限制都不应超过这个数值。通常这个系统级硬限制是Linux系统在启动时根据系统硬件资源状况计算出来的最佳的最大同时打开文件数限制，`如果没有特殊需要，不应该修改此限制，除非想为用户级打开文件数限制设置超过此限制的值`。修改此硬限制的方法是修改/etc/rc.local脚本，在脚本中添加如下行：

```bash
echo 22158 > /proc/sys/fs/file-max
```

这是让Linux在启动完成后强行将系统级打开文件数硬限制设置为22158。修改完后保存此文件。


完成上述步骤后重启系统，一般情况下就可以将Linux系统对指定用户的单一进程允许同时打开的最大文件数限制设为指定的数值。如果重启后用 ulimit-n命令查看用户可打开文件数限制仍然低于上述步骤中设置的最大值，`这可能是因为在用户登录脚本/etc/profile中使用ulimit -n命令已经将用户可同时打开的文件数做了限制`。由于通过ulimit-n修改系统对用户可同时打开文件的最大数限制时，新修改的值只能小于或等于上次 ulimit-n设置的值，因此想用此命令增大这个限制值是不可能的。所以，如果有上述问题存在，就只能去打开/etc/profile脚本文件，在文件中查找是否使用了ulimit-n限制了用户可同时打开的最大文件数量，如果找到，则删除这行命令，或者将其设置的值改为合适的值，然后保存文件，用户退出并重新登录系统即可。

通过上述步骤，就为支持高并发TCP连接处理的通讯处理程序解除关于打开文件数量方面的系统限制。

## 三、网络内核对TCP连接的显示：

1、修改网络内核对TCP连接的本地端口范围限制：

在Linux上编写支持高并发TCP连接的客户端通讯处理程序时，有时会发现尽管已经解除了系统对用户同时打开文件数的限制，但仍会出现并发TCP连接数增加到一定数量时，再也无法成功建立新的TCP连接的现象。出现这种现在的原因有多种。`第一种原因可能是因为Linux网络内核对本地端口号范围有限制`。此时，进一步分析为什么无法建立TCP连接，会发现问题出在connect()调用返回失败，查看系统错误提示消息是“Can’t assign requestedaddress”。同时，如果在此时用tcpdump工具监视网络，会发现根本没有TCP连接时客户端发SYN包的网络流量。这些情况说明问题在于本地Linux系统内核中有限制。其实，问题的根本原因在于Linux内核的TCP/ip协议实现模块对系统中所有的客户端TCP连接对应的本地端口号的范围进行了限制(例如，内核限制本地端口号的范围为1024~32768之间)。

当系统中某一时刻同时存在太多的TCP客户端连接时，由于每个TCP客户端连接都要占用一个唯一的本地端口号(此端口号在系统的本地端口号范围限制中)，如果现有的TCP客户端连接已将所有的本地端口号占满，则此时就无法为新的TCP客户端连接分配一个本地端口号了，因此系统会在这种情况下在connect()调用中返回失败，并将错误提示消息设为“Can’t assignrequested address”。内核编译时默认设置的本地端口号范围可能太小，因此需要修改此本地端口范围限制。

1）第一步，修改/etc/sysctl.conf文件，在文件中添加如下行：

```bash
net.ipv4.ip_local_port_range = 1024 65000
```

这表明将系统对本地端口范围限制设置为1024~65000之间。`请注意，本地端口范围的最小值必须大于或等于1024；而端口范围的最大值则应小于或等于65535`。修改完后保存此文件。

2）第二步，执行sysctl命令：

```bash
# sysctl -p
```

如果系统没有错误提示，就表明新的本地端口范围设置成功。如果按上述端口范围进行设置，则理论上单独一个进程最多可以同时建立60000多个TCP客户端连接。

2、修改网络内核IP_TABLE防火墙对最大跟踪的TCP连接数限制：

修改了最大文件打开数，但仍会出现并发TCP连接数增加到一定数量时，再也无法成功建立新的TCP连接的现象。`第二种无法建立TCP连接的原因可能是因为Linux网络内核的IP_TABLE防火墙对最大跟踪的TCP连接数有限制`。此时程序会表现为在 connect()调用中阻塞，如同死机，如果用tcpdump工具监视网络，也会发现根本没有TCP连接时客户端发SYN包的网络流量。由于 IP_TABLE防火墙在内核中会对每个TCP连接的状态进行跟踪，跟踪信息将会放在位于内核内存中的conntrackdatabase中，这个数据库的大小有限，当系统中存在过多的TCP连接时，数据库容量不足，IP_TABLE无法为新的TCP连接建立跟踪信息，于是表现为在connect()调用中阻塞。此时就必须修改内核对最大跟踪的TCP连接数的限制，方法同修改内核对本地端口号范围的限制是类似的：

1）第一步，修改/etc/sysctl.conf文件，在文件中添加如下行：

```bash
net.ipv4.ip_conntrack_max = 10240
```

这表明将系统对最大跟踪的TCP连接数限制设置为10240。请注意，此限制值要尽量小，以节省对内核内存的占用。

2）第二步，执行sysctl命令：

```bash
# sysctl -p
```

如果系统没有错误提示，就表明系统对新的最大跟踪的TCP连接数限制修改成功。如果按上述参数进行设置，则理论上单独一个进程最多可以同时建立10000多个TCP客户端连接。

## 【补充】优化好的内核参数sysctl.conf：

`/etc/sysctl.conf` 是用来控制linux网络的配置文件，对于依赖网络的程序（如web服务器和cache服务器）非常重要，RHEL默认提供的最好调整。推荐配置（把原/etc/sysctl.conf内容清掉，把下面内容复制进去）：

```bash
net.ipv4.ip_local_port_range = 1024 65536
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_window_scaling = 0
net.ipv4.tcp_sack = 0
net.core.netdev_max_backlog = 30000
net.ipv4.tcp_no_metrics_save=1
net.core.somaxconn = 262144
net.ipv4.tcp_syncookies = 0
net.ipv4.tcp_max_orphans = 262144
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
```

修改完毕后，执行/sbin/sysctl -p 生效。

## 四、使用支持高并发网络I/O的编程技术：

在Linux上编写高并发TCP连接应用程序时，必须使用合适的网络I/O技术和I/O事件分派机制。
可用的I/O技术有同步I/O，非阻塞式同步I/O(也称反应式I/O)，以及异步I/O。`在高TCP并发的情形下，如果使用同步I/O，这会严重阻塞程序的运转，除非为每个TCP连接的I/O创建一个线程。但是，过多的线程又会因系统对线程的调度造成巨大开销。因此，在高TCP并发的情形下使用同步 I/O是不可取的，这时可以考虑使用非阻塞式同步I/O或异步I/O。非阻塞式同步I/O的技术包括使用select()，poll()，epoll等机制。异步I/O的技术就是使用AIO。`


`从I/O事件分派机制来看，使用select()是不合适的，因为它所支持的并发连接数有限(通常在1024个以内)。如果考虑性能，poll()也是不合适的，尽管它可以支持的较高的TCP并发数，但是由于其采用“轮询”机制，当并发数较高时，其运行效率相当低，并可能存在I/O事件分派不均，导致部分TCP连接上的I/O出现“饥饿”现象`。而如果使用epoll或AIO，则没有上述问题(早期Linux内核的AIO技术实现是通过在内核中为每个 I/O请求创建一个线程来实现的，这种实现机制在高并发TCP连接的情形下使用其实也有严重的性能问题。但在最新的Linux内核中，AIO的实现已经得到改进)。

综上所述，在开发支持高并发TCP连接的Linux应用程序时，应尽量使用epoll或AIO技术来实现并发的TCP连接上的I/O控制，这将为提升程序对高并发TCP连接的支持提供有效的I/O保证。

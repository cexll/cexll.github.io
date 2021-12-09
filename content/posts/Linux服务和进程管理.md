---
title: Linux服务和进程管理
subtitle: ""
date: 2021-06-24 13:40:09.218
lastmod: 2021-06-24 13:40:09.218
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
  linux,进程
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






![LinuxFuWuHeJinChengGuanLi.png](http://192.168.1.41:8090/upload/2020/1/LinuxFuWuHeJinChengGuanLi-c3ede0c16ac546a7936e8fb97fe6cbe1.png#vwid=900&vhei=400)


<!-- more -->

## 进程管理
进程管理的三个主要任务
- 判断服务器的健康状态
- 查看所有正在运行的进程
- 强制终止进程

### 进程查看

#### ps aux        
```bash
查看当前系统所有运行的进程（可以不加-）
    -a 显示前台所有进程
    -u 显示用户名
     -x 显示后台进程
```


命令执行结果示例：
```
ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.3  41280  3732 ?        Ss   9月26   0:02 /usr/lib/systemd/systemd --switched-root --system
root         2  0.0  0.0      0     0 ?        S    9月26   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    9月26   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   9月26   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        S    9月26   0:00 [kworker/u2:0]
root         7  0.0  0.0      0     0 ?        S    9月26   0:00 [migration/0]
root         8  0.0  0.0      0     0 ?        S    9月26   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        S    9月26   0:00 [rcuob/0]
root        10  0.0  0.0      0     0 ?        S    9月26   0:36 [rcu_sched]
root        11  0.0  0.0      0     0 ?        S    9月26   0:32 [rcuos/0]
···
```

参数说明:

参数 | 说明
---|---
USER|用户名
PID |进程PID 1  init  系统启动的第一个进程
%CPU|cpu占用百分比
%MEM|内存占用百分比
VSZ |虚拟内存占用量（KB）
RSS |固定内存占有量
TTY |登录终端  tty1-7 本地终端1-6 字符、 7图形） pts/0-255
STAT|状态 （S：睡眠 D：不可唤醒 R：运行 T：停止 Z：僵死 W：进入内存交换 X：死掉的进程 <:高优先级 N：低优先级 L：被锁进内存 s：含子进程 +：位于后台 l：多线程）
START| 进程触发时间
TIME | 占用cpu时间
COMMAND | 进程本身

#### pstree                    
> * -a 查看进程树

命令执行结果示例：
```
pstree -a
systemd --switched-root --system --deserialize 21
  ├─AliHids
  │   └─4*[{AliHids}]
  ├─AliYunDun
  │   └─8*[{AliYunDun}]
  ├─AliYunDunUpdate
  │   └─3*[{AliYunDunUpdate}]
  ├─agetty --noclear tty1 linux
  ├─aliyun-service -d
  ├─crond -n
  ├─dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
  ├─memcached -d -m 128 -u root -p 11211
  │   └─6*[{memcached}]
  ├─nginx
  │   └─nginx
  ├─ntpd -u ntp:ntp -g
  ├─php-fpm
  │   ├─php-fpm
  │   └─php-fpm
  ├─rsyslogd -n
  │   └─2*[{rsyslogd}]
  ├─sshd -D
  │   └─sshd
  │       └─bash
  │           └─pstree -a
  ├─systemd-journal
  ├─systemd-logind
  └─systemd-udevd
```

#### top                    
> 实时显示进程状态

命令执行结果示例：
```
top - 15:04:52 up 2 days,  5:25,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  70 total,   2 running,  68 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1016796 total,   599400 free,    41948 used,   375448 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   838500 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
  917 root      20   0   82696   9152   5456 S  0.3  0.9   5:50.76 AliHids
    1 root      20   0   41280   3732   2388 S  0.0  0.4   0:02.36 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.01 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.00 ksoftirqd/0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    6 root      20   0       0      0      0 S  0.0  0.0   0:00.26 kworker/u2:0
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0
    ···
```

参数说明
- 第一行：系统当前时间，系统持续时间， 登录用户，1,5,15分钟之前的平均负载
- 第二行：进程总数
- 第三行：CPU占用率
- 第四行：内存使用：总共，空闲，已使用，缓存
- 第五行：swap使用情况

操作命令：
- M,按内存占用排序
- P,安CPU占用排序
- q,退出


### 终止进程

#### kill 结束单个进程
> kill命令是通过向进程发送指定的信号来结束相应进程的。在默认情况下，采用编号为15的TERM信号。TERM信号将终止所有不能捕获该信号的进程。对于那些可以捕获该信号的进程就要用编号为9的kill信号，强行“杀掉”该进程。
>
> 命令格式：kill 信号  PID

信号，进程间的通信方式


我们常用的信号有

信号名称| 信号| 意义
------|-----|-----
HUP   | 1   | 终端断线
INT   | 2   | 中断（同 Ctrl + C）
QUIT  | 3   | 退出（同 Ctrl + \）
TERM  | 15  |  终止
KILL  | 9   | 强制终止
CONT  | 18  |  继续（与STOP相反， fg/bg命令）
STOP  | 19  |  暂停（同 Ctrl + Z）

示例：结束 memcached 进程

获取memcached进程pid（24428，即为memcached进程PID）
```
ps -aux | grep memcache
root     24428  0.0  0.0 323120   864 ?        Ssl  11:00   0:02 /usr/local/memcached/bin/memcached -d -m 128 -u root -p 11211
root     24727  0.0  0.0 112664   984 pts/0    S+   15:54   0:00 grep --color=auto memcache
#
ps -ef | grep memcache
root     24428     1  0 11:00 ?        00:00:02 /usr/local/memcached/bin/memcached -d -m 128 -u root -p 11211
root     24708 24568  0 15:49 pts/0    00:00:00 grep --color=auto memcache
```
或者使用pidof查看 （ pid + of ）
```
[root@...]# pidof memcached
24428
```
终止 memcached
```
kill -9 24428
ps -aux | grep memcache
root     24729  0.0  0.0 112664   984 pts/0    S+   15:55   0:00 grep --color=auto memcache
```

#### killall   
> 杀死指定名字的进程
>
> 命令格式：killall 信号  进程名

示例：
```
killall -9 memcached
```

#### pkill
> 支持按照一定规则匹配来杀死进程
>
> 命令格式：pkill [options] <pattern>

示例：杀死用户 wahaha 下的所有进程
```
pkill -u wahaha
```
把某个终端登陆的用户踢出
```
pkill -9 -t 终端号
```
把本地登陆终端1登陆用户踢出
```
pkill -9 -t tty1                              
```

## 服务管理
### Linux中服务的分类

#### 系统默认安装的服务(RPM)
- 独立的服务
- 基于xinetd的服务，xinetd是系统超级守护进程
> xinetd服务其本身就是一个独立的服务。
>
> 当程序调用xinetd服务时，它先调用的事xinetd服务，让后xinetd服务在调用索要调用的服务进行相应。
>
> Linux系统默认是没有安装xinetd服务的，需要进行安装后才能使用。

#### 源码包安装的服务
### 系统默认安装的服务
#### 如何区分服务的分类
查看服务的自启动状态
```
chkconfig  --list                      
```
运行结果：
```
chkconfig  --list
注意：该输出结果只显示 SysV 服务，并不包含原生 systemd 服务。SysV 配置数据可能被原生 systemd 配置覆盖。
      如果您想列出 systemd 服务,请执行 'systemctl list-unit-files'。
      欲查看对特定 target 启用的服务请执行
      'systemctl list-dependencies [target]'。
aegis          	0:关	1:关	2:开	3:开	4:开	5:开	6:关
agentwatch     	0:关	1:关	2:开	3:开	4:开	5:开	6:关
netconsole     	0:关	1:关	2:关	3:关	4:关	5:关	6:关
network        	0:关	1:关	2:开	3:开	4:开	5:开	6:关
```

Linux的运行级别：0-6

级别| 说明
---|-----
0  |关机
1  |单用户模式
2  |不完全多用户，不包含NFS服务
3  |完全多用户,字符界面
4  |未分配
5  |图形界面
6  |重启

查看当前系统的运行级别：
```
runlevel
N 3
```
切换系统当前的运行级别：

命令 | 含义
----|-----
init  0 |关机                   
init  5 |切换到图形界面（前提图形界面已经安装）
init  3 |切换到字符界面
init  6 |重启

#### 独立的服务管理

- 启动

第一种方式：
```
/etc/rc.d/init.d/服务名 start| stop | restart | status
# 例：
/etc/rc.d/init.d/httpd start
```
第二种方式：（只支持RedHat系列的Linux）
```
service 服务名 tart| stop | restart | status
```
***service命令其本质是当命令运行时直接在/etc/rc.d/init.d目录下查找相应的服务，并进行相应的操作。）***

- 自启动
-
第一种方式：
```
chkconfig --level 2345 服务名 on|off
```
第二种方式：（推荐）
```
vi  /etc/rc.local (系统启动时会运行该文件)
```
修改文件内容：
```
touch /var/lock/subsys/local （更新系统的开机时间）
# 在下一行，写入自己要启动的服务名，比如我要开机自启动httpd服务：
# 就加入/etc/rc.d/init.d/httpd start
# 更改后文件就是：
touch /var/lock/subsys/local
/etc/rc.d/init.d/httpd start
```

#### ntsysv自启动管理工具
所有系统默认安装服务都可以使用ntsysv命令进行自启动管理。rpm包安装服务，自启动管理工具（只要rpm安装的，都可进行管理）

### 源码包安装的服务
启动
```
/usr/local/apache2/bin/apachectl  start
```
自启动
```
vi /etc/rc.local         
加入
/usr/local/apache2/bin/apachectl  start
```


## 计划任务
> 首先保证crond服务时启动的（crond默认是自启动的）

命令：crontab

编辑格式： * * * * *  命令

说明：
- 第一个*：一小时中第几分钟  0-59
- 第二个*：一天中第几个小时  0-23
- 第三个*：一个月中第几天    1-31
- 第四个*：一年第几个月      1-12
- 第五个*：一周中星期几       0-6             

例
```
10  *  31  *  *  命令
10  *  *  *  *  命令
5  4  *  5-10  *  命令
*/10  *  *  *  *  命令
5 4  1,15  *  *  命令  #日期和星期不要同时指定，会超出预期
5 4 10 * 5 命令
*/20 4 * 5 2   命令    #每隔二十分钟
```

查看系统定时任务
```
crontab  -l
```
删除定时任务(慎用，删除之前记得备份数据)
```
crontab  -r
```

**注意事项：**
- 选项都不能为空，必须填入，不知道的值使用通配符*表示任何时间
- 每个时间字段都可以指定多个值，不连续的值用,间隔，连续的值用-间隔
- 间隔固定时间执行书写为*/n格式
- 命令应该给出绝对路径
- 星期几何第几天不能同时出现
- 最小时间范围是分钟，最大时间范围是月


## 查看系统启动信息
查看系统启动信息
```
dmesg
```
系统启动信息日志
```
cat  /var/log/dmesg
```
查看eth0信息
```
dmesg | grep eth0                   
```
查看cpu信息
```
dmesg | grep CPU                   
```

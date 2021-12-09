---
title: Mysql5.7及以上忘记密码如何解决
subtitle: ""
date: 2021-06-24 13:38:11.452
lastmod: 2021-06-24 13:38:11.452
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    mysql,忘记密码
]
categories: [
    mysql
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






# 解决MySQL新版本更改密码等操作


检查MySQL是否运行：

```bash
sudo netstat -tap | grep mysql
```

如果成功安装，我的会显示如下内容：

```bash
tcp        0      0 localhost:mysql         *:*                     LISTEN      18475/mysqld 
```

PS：重启/打开/关闭MySQL的方法是：`sudo service mysql restart/start/stop`

就这两个命令就安装好了，可是我在安装过程中并没有出现要我写用户名和密码的地方，我一脸懵逼，
完成后在终端输入mysql -u root -p之后，要求我输入密码，可是我并不知道密码，随便输入之后，

```bash
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

百度了三五个小时，解决方案五花八门，我最后使用有效的方法是：
打开一个文件

```bash
sudo vim /etc/mysql/debian.cnf
```

在这个文件里面有着MySQL默认的用户名和用户密码，
最最重要的是：用户名默认的不是root，而是debian-sys-maint，如下所示

```bash
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = hGu99nJgoWcmCDKT
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = hGu99nJgoWcmCDKT
socket   = /var/run/mysqld/mysqld.sock
basedir  = /usr
```

密码会随即给一个很复杂的，这个时候，要进入MySQL的话，就是需要在终端把root更改为debian-sys-maint，如下代码

```bash
mysql -u debian-sys-maint -p 
然后终端会提示你输入密码
Enter password:  
```

这是输入文件中的密码即可成功登陆。
当然了，这之后就要修改密码了，毕竟密码太难记。

经过度娘的指导，我所安装的版本是5.7，所以password字段已经被删除，取而代之的是authentication_string字段，所以要更改密码：

```bash
mysql> use mysql;
mysql> update user set authentication_string=PASSWORD("自定义密码") where user='root';
mysql> update user set plugin="mysql_native_password";
mysql> flush privileges;
mysql> quit;
```

如果显示：

```bash
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1
```

则代表成功修改，之后需要*重启**MySQL，方可登录成功。
顺便说一下删除MySQL的方法，省的之后再找度娘。
代码如下:

```bash
sudo apt-get autoremove --purge mysql-server-5.7
sudo apt-get remove mysql-server
sudo apt-get autoremove mysql-server
sudo apt-get remove mysql-common
```

上面的可能会有些是多余的，之后需要清理残余数据

```bash
 dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P
```
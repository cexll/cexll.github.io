---
title: Ubuntu编译安装PHP7
subtitle: ""
date: 2021-06-24 13:28:11.601
lastmod: 2021-06-24 13:28:11.601
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    linux,编译,php,ubuntu
]
categories: [
    linux,php
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




# 第一步 下载PHP源代码 
在 `https://php.net/` 下载最新稳定安装包,现在最新版是 `PHP7.4.11` , 我们下载xz格式的, `https://www.php.net/distributions/php-7.4.11.tar.xz`
下载到`download`目录
# 第二步 解压
```bash
> tar xvf php-7.4.11.tar.xz
> cd php-7.4.11
```

# 第三步 安装各种lib
为系统安装需要的`lib`,这里是一把梭安装命令,Ubuntu18.04以上适用
```bash
sudo apt install gcc make openssl curl libbz2-dev libxml2-dev libjpeg-dev libpng-dev libfreetype6-dev libzip-dev build-essential pkg-config  libsqlite3-dev libssl-dev libzip-dev bison autoconf build-essential pkg-config \
libltdl-dev libbz2-dev libxml2-dev libxslt1-dev libssl-dev libicu-dev libpspell-dev \
libenchant-dev libmcrypt-dev libpng-dev libjpeg8-dev libfreetype6-dev libmysqlclient-dev \
libreadline-dev libcurl4-openssl-dev librecode-dev libsqlite3-dev libonig-dev zip
```
# 第四步 configure 
这里使用了一把梭配置脚本,看看不需要什么,需要什么,加上就行了
```bash
./configure --prefix=/usr/local/php7.4 --with-config-file-path=/usr/local/php7.4/etc --with-config-file-scan-dir=/usr/local/php7.4/conf.d --with-mysql --with-mysqli --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --enable-discard-path --enable-magic-quotes --enable-safe-mode --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-fastcgi --enable-fpm --enable-force-cgi-redirect --enable-mbstring --with-mcrypt --enable-ftp --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --with-gettext --with-mime-magic --with-pear 
```

# 第五步 编译 and 安装
```bash
> make -j 10(这里值你的CPU核心)
> sudo make install
安装好之后,将当前目录下`php.pro...`复制到 `/usr/local/php7.4/etc/php.ini`,此乃你的`php.ini`配置文件,这里的路径根据你的为准
```

# 第六步 配置变量
`php`是安装好了,但是要让系统知道你安装了`php`在什么位置,我这里推荐使用环境变量配置
```bash
> export PATH=$PATH:/usr/local/php7.4/bin
执行完这一行之后, 在任何地方,执行 `php -v` 即可运行php, 但是关闭终端之后就没有了,这怎么办呢
这里推荐两个方法
1. vim ~/.bashrc 
编辑当前用户的配置,你如果安装了`zsh`,配置了`ohmyzsh` 那么,你修改 `zshrc` 文件即可
2. vim /etc/profile 
编辑全局配置
```

# 完结 
全部配置好之后,就可以愉快的玩耍了,后续安装好 `composer` , `pecl` 来安装各种扩展 ,舒服的一批啊
```bash
PHP 7.4.10 (cli) (built: Sep 19 2020 22:33:12) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.10, Copyright (c), by Zend Technologies
```
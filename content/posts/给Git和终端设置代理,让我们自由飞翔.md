---
title: 给Git和终端设置代理,让我们自由飞翔
subtitle: ""
date: 2021-06-24 13:37:02.903
lastmod: 2021-06-24 13:37:02.903
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    git,proxy
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




# 给Git和终端设置代理,让我们自由飞翔

```bash
Set up agents for git and the terminal
```

* 终端设置只适用于Linux/MacOS,Windows不行,Git通用

# 给终端设置代理

首先,电脑要先下载类SS/SSR等软件,或者自己有VPS服务器,由于每一款代理不一样,有的是http或者socks5,输入指令前自己了解一下

```bash
# http
export http_proxy=http://proxyAddress:port
# socks5
export ALL_PROXY=socks5://proxyAddress:port
```

例:

```bash
# 由于我是SSR,所以使用socks5演示
export ALL_PROXY=socks5://127.0.0.1:1080
```

这样输入是设置的临时代理,如果只是临时设置那么可以这样,如果我想每次开机都代理呢? 可以这样

```bash
$ vim ~/.bashrc # 提示没有vim的先安装
# 在最后一行添加上面的命令,我这里只做演示,每个人不一样自己注意
...
export ALL_PROXY=socks5://127.0.0.1:1080
```

# 给Git设置代理

给Git设置代理后下载/上传不会受墙限制

```bash
git config  --global http.proxy http://user:pwd@server.com:port
```
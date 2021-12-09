---
title: 使用Python运行FTP服务器
subtitle: ""
date: 2021-06-24 13:37:49.652
lastmod: 2021-06-24 13:37:49.652
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    python,ftp
]
categories: [
    python
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




# 使用Python搭建FTP服务器

### 首先确认已经安装了`Python`,`PIP`,没有安装的请自行安装

```bash
# 在确认有Python和pip环境后使用PIP安装FTP库
$ sudo pip install pyftpdlib
# 安装完成后运行它
$ python -m pyftpdlib
# 这是运行后的界面 
[I 2018-12-23 00:55:15] >>> starting FTP server on 0.0.0.0:2121, pid=27 <<<
[I 2018-12-23 00:55:15] concurrency model: async
[I 2018-12-23 00:55:15] masquerade (NAT) address: None
[I 2018-12-23 00:55:15] passive ports: None
```

这个使用在本地电脑或者同一个网络环境下的电脑或者手机,打开浏览器输入运行了 `pyftpdlib` 电脑的 `ftp://IP地址 + 2121` 例: `ftp://192.168.1.10:2121`

**然后我们就可以在本地网络下互相下载文件了 :)**
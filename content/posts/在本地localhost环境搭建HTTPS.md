---
title: 在本地localhost环境搭建HTTPS
subtitle: ""
date: 2021-06-24 13:30:40.6
lastmod: 2021-06-24 13:30:40.6
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    https,ssl,localhost
]
categories: [
    nginx
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



 今天闲着,水一篇文章

# 使用 `mkcert` 生成SSL证书,在本地运行HTTPS

在开发过程中,有些会需要到HTTPS,这就比较麻烦了,又不可能一直使用`FTP/SFTP`同步代码,个人认为,在本地开发还是更爽一点

所以今天就做(水)一篇教程,在本地运行HTTPS

# Download

使用 `mkcert` , 下载地址 https://github.com/FiloSottile/mkcert

# Run

下载到本地之后, 配置到环境变量, Windows 在设置,环境变量PATH里面添加`mkcert`目录, Linux/Macos 使用 `export`添加环境变量, 或者移动到`/usr/bin/mkcert`,直接使用 `mkcert` 运行

```bash
mkcert -install example.com (例: cwj0.top) 生成证书, 自动安装
| mkcert -install example.com (例: cwj0.top) 生成证书, 我们可以生成证书,不自动安装
```

运行命令之后会在本地生成`example.com.pem` and `example.com-key.pem`

这样就算完成啦, 如果是某些软件需要HTTPS,配置好证书地址就ok, 如果是WEB环境,在`nginx`或者`apache`配置好SSL`and` SSL_PATH 就行 

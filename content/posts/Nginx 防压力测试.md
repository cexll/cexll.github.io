---
title: Nginx 防压力测试
subtitle: ""
date: 2021-06-24 13:31:34.399
lastmod: 2021-06-24 13:31:34.399
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    nginx,压力测试,dd
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




# 直接简单的方法限制同一个IP的并发最大为10

## 在`nginx.conf`编辑启用流量/IP/并发限制
```nginx
vim nginx.conf
limit_conn_zone $binary_remote_addr zone=perip:10m; # 启用IP限制
limit_conn_zone $binary_remote_addr zone=one:10m; # 启用并发限制
limit_conn_zone $server_name zone=perserver:10m; 
```

## 然后在要防止的网站配置
```nginx
limit_conn perserver 300; # 并发限制
limit_conn perip 25; # 单IP限制
limit_rate 512k; # 流量限制
```
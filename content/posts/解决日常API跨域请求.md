---
title: 解决日常API跨域请求
subtitle: ""
date: 2021-06-24 13:38:48.477
lastmod: 2021-06-24 13:38:48.477
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    nginx,跨域
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




# 通过Nginx设置

```nginx
# 在location下配置，配置看需求而定
location / {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,Keep-Alive,User-Agent,Cache-Control,Content-Type,Authorization';
}
```

# PHP 设置

*php 只需要在入口除设置好header就行*

```php
header('Access-Control-Allow-Origin: *'); // 允许任意域名发起的跨域请求
header('Access-Control-Allow-Headers: X-Requested-With,X_Requested_With');
header('Access-Control-Allow-Methods: GET,POST,PUT');
```

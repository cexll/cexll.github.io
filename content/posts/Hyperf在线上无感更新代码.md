---
title: Hyperf在线上无感更新代码
subtitle: ""
date: 2021-06-24 13:27:27.981
lastmod: 2021-06-24 13:27:27.981
draft: true
author: "cexll"
authorLink: ""
description: ""

tags: [
    hyperf,php
]
categories: [
    php,hyperf
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




# 使用hyperf因为2.1有缓存所以更新代码都会扫描一次，我们可以用这个思路让他先扫描在重启

在线上`.env`编辑两项参数 `APP_ENV` 默认为 `dev`, `SCAN_CACHEABLE` 可以在文档找到, (是否使用注解扫描缓存)
```.env
APP_ENV=prod
SCAN_CACHEABLE=true
```
编辑好之后我们模拟一下代码更新场景
```bash
git push origin master // 本地push代码
// 线上环境
git pull origin master // 我们拿到最新代码
composer dump-autoload -o // 删除缓存
php bin/hyperf.php // 刷新缓存
supervisorctl restart hyperf // 重启服务
```

使用以上操作就可以无感更新代码
这几项理论上都是可以连着操作 git ssh 设置好密钥,像这样
```bash
git pull origin master && composer dump-autoload -o && && php bin/hyperf.php && supervisorctl restart hyperf
```